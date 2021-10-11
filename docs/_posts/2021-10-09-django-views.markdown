---
layout: single
title:  "Django - Views"
date:   2021-10-09 07:30:00 +1000
toc: true
categories: python django views
---

A view in django can be thought of a request handler.  The term "view" can be misleading.

## Creating a View

To create a view define a function in `views.py` of the app that will take a request and return some data.

{% highlight python %}
from django.shortcuts import render
from django.http import HttpResponse

def get_token(request):

    return HttpResponse('mytoken')
{% endhighlight %}

## Mapping a View to a URL

To allow clients to request the view it needs to be mapped to a URL.  This is done in `urls.py`.  The below code creats a custom `urls.py` in the app directory.

{% highlight python %}
# urls.py in app

from django.urls import path
from . import views

urlpatterns = [
    path('token/', views.get_token),
]
{% endhighlight %}

To use the app `urls.py` we need to include in in the project level `urls.py`.  Need to add `include` to the imports.

{% highlight python %}
# urls.py in project

from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('auth/', include('jwtauth.urls'))
]
{% endhighlight %}

## Returning JSON Data

JSON objects can be returned (by default must be a dictionary object unless `safe` parameter is specified as `False`).

{% highlight python %}
from django.shortcuts import render
from django.http import HttpResponse
from django.http import JsonResponse


def get_token(request):

    return HttpResponse('mytoken')

def get_json_token(request):

    return JsonResponse({"token":"mytoken"})

{% endhighlight %}


