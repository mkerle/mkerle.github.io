---
layout: single
title:  "Django Custom Authentication with LDAP and JWT"
date:   2022-06-05 07:30:00 +1000
toc: true
categories: django api python rest authentication ldap jwt
---

This guide is an alternative to using the built authentication backends that are available in Django.

The reason for doing a custom authenction method is:
- It was not obvious how to integrate remote LDAP authentication with JWTs
- The django documentation seemed to miss some details on how to glue the components together
- The django documentation seemed to rely a lot on User models in the backend which may not be necessary if Django is used as API REST backend

Below is a PoC to get remote LDAP authentication while utilising JWTs to authorise requests to Django API REST targets.

# Creating the LDAP JWT Authentication Module

To perform the remote LDAP authentication and return \ validate JWT we need to create a new module.  This can be done in the root of the Django project or anywhere else on the import path.

## Install Required Modules
To start we will need some additional modules.  The below could be used if pip virtual environment is used (`pipenv shell`).

NOTE: If using on windows it is likely to run into build errors for `python-ldap` so there are pre-compiled wheel available for `python-ldap` at [here][python-ldap-wheel].  Make sure to select the wheel that corresponds with the version of python that is running.

{% highlight bash %}
$ pip install Authlib
$ pip install python-ldap
$ pip install cryptography
{% endhighlight %}

## Create the Class


{% highlight python %}
import json
import datetime
from authlib.jose import jwt
from cryptography.hazmat.primitives import serialization
import ldap

class ldapJWTAuthenticator():

    LDAP_SERVER_IP = '192.168.1.10'
    LDAP_SERVER_PORT = '389'
    LDAP_BASE_DN = 'cn=users,dc=mydomain,dc=zone'
    LDAP_BIND_DN = 'ldap_bind_acc'
    LDAP_BIND_PASSWORD = 'MyBindAccountPassword'
    LDAP_TLS_ENABLE = False
{% endhighlight %}

- `LDAP_SERVER_IP` is the IP address of LDAP\ Active Directory server
- `LDAP_SERVER_PORT` is the port that LDAP is listening on.  389 is default for non-tls.
- `LDAP_BASE_DN` is the base DN in LDAP where users will need to be searched\ authenticated.
- `LDAP_BIND_DN` is the DN of the account used to perform the initial search in LDAP
- `LDAP_TLS_ENABLE` is a flag to indicate if LDAP of TLS is to be used (FUTURE USE).


## Authenticating against LDAP

Add the below methods to perform the remote authencitation against LDAP.

The first method required is `ldapSearchUser()`.  This method will search LDAP using the bind account to find users with `sAMAccountName = user`.  If the user is found it will then attempt to bind to LDAP using the `DN` (obtained from `getUserDN()`) of `user` as well as the specified `password`.

{% highlight python %}
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

    def getUserDN(self, userData):

        dn = ''
        if (userData and len(userData) == 1 and userData[0][1] and 'distinguishedName' in userData[0][1]):
            return userData[0][1]['distinguishedName'][0].decode('utf-8')

        return dn
{% endhighlight %}


Next we will add some wrapper methods.  `authenticate()` is the main method that will be called by a login endpoint while `ldapAuthenticate()` is looks at the returned LDAP data for the user to determine if it is valid.

{% highlight python %}
    def authenticate(self, username=None, password=None):

        if (username is not None or password is not None):

            ldapResult = self.ldapSearchUser(username, password)
            if (self.ldapAuthenticate(ldapResult)):
                return ldapResult[0][1]

        return None

    def ldapAuthenticate(self, ldapSearchResult):

        if (ldapSearchResult is not None):
            if (len(ldapSearchResult) == 1):
                if (ldapSearchResult[0][1]):
                    return True

        return False

{% endhighlight %}


## JWT Certificates

Before we can add our JWT code we will need a public and private certificate to ensure the integrity of JWTs.  The public certificate can be used by externall applications to verify a JWT.  The private key is kept on the server for encrption of keys and validation of requests.

For a PoC we can generate a public and privte key pair using the below command with `openssl`.

{% highlight bash %}
$ openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -days 365
{% endhighlight %}



## JWT Setup

In our module we will add some class members for JWT parameters.

{% highlight python %}
    ...

    JWT_ISSUER = 'jwt.mitch.zone'
    JWT_DEFAULT_EXPIRY = 60     # seconds
    JWT_PRIVATE_KEY_PATH = './key.pem'
    JWT_PRIVATE_KEY_PASSWORD = 'Password1'      # retrieve from keyring or at least obfuscate
    JWT_PUBLIC_KEY_PATH = './cert.pem'

    ...
{% endhighlight %}

We will add some methods to allow a view to obtain a serialised version of the public key.

{% highlight python %}

    def getPublicKey(self):

        pubKey = self.getJWTPublicKey(self.JWT_PUBLIC_KEY_PATH).public_bytes(serialization.Encoding.PEM, serialization.PublicFormat.SubjectPublicKeyInfo)
        return pubKey

    def getJWTPublicKey(self, jwtPublicKeyPath):
        with open(jwtPublicKeyPath, "rb") as key_file:
            public_key = serialization.load_pem_public_key(
                key_file.read()
            )

        return public_key

{% endhighlight %}

Finally add methods to generate and return a JWT from a users LDAP data.

{% highlight python %}
    def generateJWTExpiryTime(self, tokenExpirySecs):

        return int((datetime.datetime.now()+datetime.timedelta(seconds=tokenExpirySecs)).timestamp())

    def getJWTPrivateKey(self, jwtPrivateKeyPath, jwtPrivateKeyPassword):
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

        s = jwt.encode(header, payload, self.getJWTPrivateKey(self.JWT_PRIVATE_KEY_PATH, self.JWT_PRIVATE_KEY_PASSWORD))

        return { 'token' : s.decode('utf-8') }

{% endhighlight %}

## Adding JWT Validation

{% highlight python %}
    def validateUserJWT(self, token):

        try:
            claims_options = { 'iss' : { 'essential' : True, 'values' : [self.JWT_ISSUER]}, 'exp' : { 'essential' : True, 'validate' : self.jwtExpired }}

            claims = jwt.decode(token, self.getJWTPrivateKey(self.JWT_PRIVATE_KEY_PATH, self.JWT_PRIVATE_KEY_PASSWORD), claims_cls=JWTClaims, claims_options=claims_options)

            return claims.validate()
        except:
            return False

    def jwtExpired(exp):

        try:
            return (int(datetime.datetime.now().timestamp()) > datetime.datetime.fromtimestamp(exp))
        except:
            return False
{% endhighlight %}

# Example View Function for Login

A view function to perform the login might look like the below.

{% highlight python %}
@api_view(['GET', 'POST'])
def login(request):

    if (request.method == 'POST'):    

        data = json.loads(request.body)
        if ('username' in data and 'password' in data):

            username = data['username']
            password = data['password']

            authenticator = ldapJWTAuth.ldapJWTAuthenticator()

            userData = authenticator.authenticate(username, password)
            if (userData is not None):

                jwtToken = authenticator.generateUserJWT(userData)

                return Response(jwtToken)

            else:

                return generate401Response()

    else:

        generate401Response()
{% endhighlight %}






[python-ldap-wheel]: https://www.lfd.uci.edu/~gohlke/pythonlibs/#python-ldap