---
layout: single
title:  "Django Remote Authentication with LDAP"
date:   2022-06-12 15:00:00 +1000
toc: true
categories: django api python rest authentication ldap jwt web token
---

It is possible to use alternative authentication backends in Django other then the default local authentication.  This PoC will show how to combine remote LDAP authentication with Json Web Tokens.

# Background Info

[Authentication in Web requests][auth-web-requests] provides some good intro documentation for authenticating web requests in Django.

[Authentication Backends][auth-backends] is another good resource for how to deal with alternative backends.

[Remote User Authentication][remote-user] details how to change the authentication backend to remote authentication in django.

[Customising Django Authentication][django-auth-custom] for details on how to customise the authentication backend.

# PoC Overview

The PoC will involve:
- Custom User `LDAPJWTUser` class
- Custom Middleware for extracting JWTs from requests
- Custom LDAP authentication backend
- A JWT helper class to generate and validate JWTs

The `LDAPJWTUser` class must exist in an app folder under the Django project and placed in the `models.py`. [Custom User models][django-custom-user-model] also require updating the `AUTH_USER_MODEL` in `settings.py`.

The JWT middleware is responsible for checking requests for the `AUTHORIZATION` header and that it contains `Bearer` along with a token.

The custom LDAP authentication backend will handle the initial login that takes username and password to bind and query LDAP.

A JWT helper class with provide some features to generate the JWT as well as validate the token on incoming requests.

# Configuration

## Create the Custom User Model

Custom user models must be placed in `models.py` of an app in the Django project.  A simple custom user model for combining LDAP and JWTs is shown below.

The `USERNAME_FIELD` is required and an exception will be thrown if not specified.  Note also that an import for the `JWTHelper` is included but will be defined later.

{% highlight python %}
from django.db import models
from django.contrib.auth.models import AbstractBaseUser
from testapp.jwt import JWTHelper

class LDAPJWTUser(AbstractBaseUser):

    username = models.TextField(verbose_name='username', max_length=255, unique=True)

    USERNAME_FIELD = 'username'

    jwt = None
    LDAPGroups = []

    def setUsername(self, username):
        self.username = username

    def setJWT(self, jwt):
        self.jwt = jwt

    def setLDAPGroups(self, groupList):
        self.LDAPGroups = groupList

    def has_perm(self, perm, obj=None):

        return perm in self.LDAPGroups

    def isTokenValid(self):

        if (self.jwt is not None):

            return JWTHelper.validateUserJWT(self.jwt)

        return False
{% endhighlight %}

## Create LDAP Authentication Backend

This authentication backend will use LDAP and return the groups that a user belongs to so we can control access to endpoints.

The important point to note here is the use of `get_user_model()` so that we do not reference our custom user model directly.

{% highlight python %}
from django.contrib.auth import get_user_model
from django.contrib.auth.backends import BaseBackend
import ldap

UserModel = get_user_model()

class LDAPBackend(BaseBackend):

    LDAP_SERVER_IP = '192.168.1.10'
    LDAP_SERVER_PORT = '389'
    LDAP_BASE_DN = 'cn=users,dc=lab,dc=zone'
    LDAP_BIND_DN = 'ldap_bind_acc'
    LDAP_BIND_PASSWORD = 'bindpassword'
    LDAP_TLS_ENABLE = False

    def authenticate(self, request, username=None, password=None):

        if (username is not None or password is not None):

            ldapResult = self.ldapSearchUser(username, password)
            if (self.ldapAuthenticate(ldapResult)):

                groups = []
                for group in ldapResult[0][1].get('memberOf', []):
                    groups.append(group.decode('utf-8'))

                user = UserModel()
                user.setLDAPGroups(groups)
                print(user.LDAPGroups)
                return user

        print('returning from authenticate with none')
        return None

    def ldapSearchUser(self, user, password):

        try:
            constr = 'ldap://' + self.LDAP_SERVER_IP + ':' + self.LDAP_SERVER_PORT
            if (self.LDAP_TLS_ENABLE):
                constr = 'ldaps://' + self.LDAP_SERVER_IP + ':' + self.LDAP_SERVER_PORT

            con = ldap.initialize(constr, bytes_mode=False)

            if (self.LDAP_TLS_ENABLE):
                con.set_option(ldap.OPT_X_TLS_CACERTFILE, self.LDAP_TLS_CERT_PATH)
                con.set_option(ldap.OPT_X_TLS_NEWCTX, 0)
                con.start_tls_s()

            con.simple_bind_s(self.LDAP_BIND_DN, self.LDAP_BIND_PASSWORD)

            userLDAPData = con.search_s(self.LDAP_BASE_DN, ldap.SCOPE_SUBTREE, u'(sAMAccountName=' + user + ')')
            con.unbind_s()

            # bind using user DN and password - if wrong will throw exception
            userDN = self.getUserDN(userLDAPData)
            con = ldap.initialize(constr, bytes_mode=False)
            con.simple_bind_s(userDN, password)

            return userLDAPData
        except:
            return None

    def ldapAuthenticate(self, ldapSearchResult):

        if (ldapSearchResult is not None):
            if (len(ldapSearchResult) == 1):
                if (ldapSearchResult[0][1]):
                    return True

        return False

    def getUserDN(self, userData):

        dn = ''
        if (userData and len(userData) == 1 and userData[0][1] and 'distinguishedName' in userData[0][1]):
            return userData[0][1]['distinguishedName'][0].decode('utf-8')

        return dn
{% endhighlight %}

In the above example only remote users will be able to authenticate.  If a combination of local and remote users will need to authenticate then add `django.contrib.auth.backends.ModelBackend` to the list.

## Add JWT Helper Class

For this PoC the JWT Helper class and JWT Middleware will be in the same module.  The `JWTHelper` will do all the generation of JWTs as well as decoding and validation.

{% highlight python %}
class JWTHelper():

    JWT_ISSUER = 'jwt.lab.zone'
    JWT_DEFAULT_EXPIRY = 60     # seconds
    JWT_PRIVATE_KEY_PATH = 'testapp/key.pem'
    JWT_PRIVATE_KEY_PASSWORD = 'privatekeypassword'      # retrieve from keyring or at least obfuscate
    JWT_PUBLIC_KEY_PATH = 'testapp/cert.pem'

    def getPublicKey(self):

        pubKey = self.getJWTPublicKey(self.JWT_PUBLIC_KEY_PATH).public_bytes(serialization.Encoding.PEM, serialization.PublicFormat.SubjectPublicKeyInfo)
        return pubKey

    def getJWTPublicKey(self, jwtPublicKeyPath):
        with open(jwtPublicKeyPath, "rb") as key_file:
            public_key = serialization.load_pem_public_key(
                key_file.read()
            )

        return public_key

    def generateJWTExpiryTime(self, tokenExpirySecs):

        return int((datetime.datetime.now()+datetime.timedelta(seconds=tokenExpirySecs)).timestamp())

    @classmethod
    def getJWTPrivateKey(cls, jwtPrivateKeyPath, jwtPrivateKeyPassword):
        with open(jwtPrivateKeyPath, "rb") as key_file:
            private_key = serialization.load_pem_private_key(
                key_file.read(),
                password=bytes(jwtPrivateKeyPassword, 'utf-8'),
            )

        return private_key

    def generateUserJWT(self, userLDAPData):

        groups = []
        for group in userLDAPData.get('memberOf', []):
            groups.append(group.decode('utf-8'))

        principalName = userLDAPData.get('userPrincipalName')[0].decode('utf-8')
        name = userLDAPData.get('name')[0].decode('utf-8')

        exp = self.generateJWTExpiryTime(self.JWT_DEFAULT_EXPIRY)
        payload = {
            'iss': self.JWT_ISSUER, 
            'sub': principalName, 
            'exp' : exp, 
            'name' : name, 
            'groups' : groups  
        }
        header = {'alg': 'RS256'}

        s = JsonWebToken.encode(header, payload, self.getJWTPrivateKey(self.JWT_PRIVATE_KEY_PATH, self.JWT_PRIVATE_KEY_PASSWORD))

        return { 'token' : s.decode('utf-8') }

    @classmethod
    def validateUserJWT(cls, token):

        try:
            claims_options = { 'iss' : { 'essential' : True, 'values' : [cls.JWT_ISSUER]}, 'exp' : { 'essential' : True, 'validate' : cls.jwtExpired }}

            jwt = JsonWebToken()
            claims = jwt.decode(token, cls.getJWTPrivateKey(cls.JWT_PRIVATE_KEY_PATH, cls.JWT_PRIVATE_KEY_PASSWORD), claims_cls=JWTClaims, claims_options=claims_options)

            return claims.validate()
        except:
            return False

    def jwtExpired(exp):

        try:
            return (int(datetime.datetime.now().timestamp()) > datetime.datetime.fromtimestamp(exp))
        except:
            return False

    @classmethod
    def decodeJWT(cls, token):

        jwt = JsonWebToken()
        return jwt.decode(token, cls.getJWTPrivateKey(cls.JWT_PRIVATE_KEY_PATH, cls.JWT_PRIVATE_KEY_PASSWORD))
{% endhighlight %}


## The JWT Middleware

The JWT middleware simply checks for the `AUTHORIZATION` header and extracts the `Bearer` token.  It then decodes the JWT so that the `request.user` can be populated for further processing on a view etc.


{% highlight python %}
from django.utils.deprecation import MiddlewareMixin
from django.contrib.auth import get_user_model

class JWTAuthenticationMiddleware(MiddlewareMixin):

    def process_request(self, request):

            if ('Authorization' in request.headers):
                if ('Bearer' in request.headers['Authorization']):
                    request.user = self._getUserFromJWT(request.headers['Authorization'].replace('Bearer ', ''))


    def _getUserFromJWT(self, jwt):        

        decodedJWT = JWTHelper.decodeJWT(jwt)
        user = get_user_model()()
        user.setLDAPGroups(decodedJWT['groups'])
        user.setJWT(decodedJWT)

        return user
{% endhighlight %}



## Configuring settings.py

Now that all our backend code is done we need to update `settings.py` to use it.

Since we are using a custom user model, we must add the app that the user model is in to the `INSTALLED_APPS` setting.

{% highlight python %}
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'rest_framework',
    'testapp',
]
{% endhighlight %}

Update `MIDDLEWARE` to include the custom JWT Middleware:

{% highlight python %}
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'testapp.jwt.JWTAuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]
{% endhighlight %}

Set `AUTHENTICATION_BACKENDS` to use our LDAP authentication backend.

{% highlight python %}
AUTHENTICATION_BACKENDS = [
    'testapp.ldapBackend.LDAPBackend',
]
{% endhighlight %}

Finally, set `AUTH_USER_MODEL` to our custom user model that was defined in `models.py` for the app `testapp`.  Note that the required syntax here is `app.classname` (i.e not a path).

{% highlight python %}
AUTH_USER_MODEL = 'testapp.LDAPJWTUser'
{% endhighlight %}


## Test Views

Before we can test we need some views to perform the LDAP authentication and another where we can test that a user is authenticated and has required permissions to request the endpoint.

### Login Endpoint

The code below has not been fully implemented and is still in development.

{% highlight python %}
@api_view(['GET', 'POST'])
def login2(request):

    if (request.method == 'POST'):    

        data = json.loads(request.body)
        if ('username' in data and 'password' in data):

            username = data['username']
            password = data['password']

            user = authenticate(request, username=username, password=password)

            return Response({'result' : 'did authentication', 'groups' : user.LDAPGroups})

    return generate401Response()
{% endhighlight %}

## Permission Test Endpoint

The code below is a simple test to check for valid JWT and that a user has permissions.  Still in development.

{% highlight python %}

@api_view(['GET'])
def permTest(request):

    if request.user.is_authenticated:
        if (request.user.has_perm('CN=DUMMY_GROUP,CN=Users,DC=mitch,DC=zone')):
            return Response({'result' : 'got the data'})
        else:
            return Response({'result' : 'authenticated but no permission'})

    else:
        return Response({'result' : 'not authenticated'})

{% endhighlight %}

# Testing

Use curl to request login with credentials and receive a JWT.

{% highlight bash %}
$ curl http://192.168.1.10/testapp/login/ -X POST -d '{"username" : "testuser", "password" : "password" }'
{% endhighlight %}

To test the permissions enpoint use the returned token in the `AUTHORIZATION` header.

{% highlight bash %}
$ curl http://192.168.1.10/testapp/permtest/ -H "Authorization: Bearer <bearer token>"
{% endhighlight %}

[auth-web-requests]: https://docs.djangoproject.com/en/4.0/topics/auth/default/#auth-web-requests
[auth-backends]: https://docs.djangoproject.com/en/4.0/topics/auth/customizing/#authentication-backends
[remote-user]: https://docs.djangoproject.com/en/4.0/howto/auth-remote-user/
[django-auth-custom]: https://docs.djangoproject.com/en/4.0/topics/auth/customizing/

[django-custom-user-model]: https://docs.djangoproject.com/en/4.0/topics/auth/customizing/#substituting-a-custom-user-model

[python-ldap]: https://www.python-ldap.org/en/python-ldap-3.3.0/index.html
[python-ldap-wheel]: https://www.lfd.uci.edu/~gohlke/pythonlibs/#python-ldap