---
layout: single
title:  "Angular JWT Authentication"
date:   2021-10-12 17:30:00 +1000
toc: true
categories: angular authentication python django
---

This guide shows how to build an Angular app with JSON Web Tokens (JWT).  A backend service built using django will be implemented to provide the authentication server as well as provide JSON Web Key (JWS) URL to supply the public key so that clients can validate the JWT token.

The angular app will use angular materials to apply styles.

## Creating the Angular App

Start by creating a new angular project and add angular materials.  Accept defaults for all questions.  

{% highlight bash %}
$ ng new angular-jwt
$ cd angular-jwt
$ ng add @angular/material
{% endhighlight %}

## Create Login Component

Create the new login component using the angular cli.

{% highlight bash %}
$ ng generate component auth
{% endhighlight %}

## Create Auth Service

Create the auth service using the angular cli.  Add [`angular2-jwt`][angular-jwt] library.

{% highlight bash %}
$ ng generate service auth
$ npm install @auth0/angular-jwt
{% endhighlight %}

## Create HTTP Interceptors

To support the login to the backend we need a number of HTTP Interceptors.

- Default No Op Interceptor
- Credentials Interceptor - ensure the `withCredentials` field is set.
- Auth Interceptor - adds the JWT token to backend requests
- CSRF Interceptor - For HTTP POST method requests applies the `X-CSRFTOKEN` header.  This needs to be done manually as Angular does not include this by default when using absolute URLs (relative URLs should be automatic).  Refer to [Angular GitHub issue][github-csrf-issue] for more information. 

{% highlight bash %}
$ mkdir interceptors
$ ng generate interceptor interceptor noop
$ ng generate interceptor interceptor credentials
$ ng generate interceptor interceptor auth
$ ng generate interceptor interceptor csrf
{% endhighlight %}

## Create Django Backend

Django does not provide support for [CORS][mozilla-cors] headers out of the box.  Use the [Django CORS][django-cors] app available on GitHub.

{% highlight bash %}
$ pip install Authlib
$ pip install django-cors-headers
{% endhighlight %}


[angular-jwt]: https://github.com/auth0/angular2-jwt
[mozilla-cors]: https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS
[django-cors]: https://github.com/adamchainz/django-cors-headers
[github-csrf-issue]: https://github.com/angular/angular/issues/18859