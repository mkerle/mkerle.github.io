---
layout: single
title:  "Django API - Getting Started"
date:   2022-06-04 16:35:00 +1000
toc: true
categories: django api python rest
---

Django includes a framework for building REST APIs.

# Creating a Django API Development Environment

The initial process is similar to creating a django project where we create a python virtual environment.  Use the below when creating a new project from scratch.  Otherwise `pipenv install djangorestframework` can be used in an existing django project.

{% highlight bash %}
$ python -m pip install --upgrade pip
$ pip install pipenv

$ cd ~/<somedir>
$ mkdir <project>
$ cd <project>

$ pipenv install djangorestframework

$ pipenv shell
$ pipenv --venv
$ django-admin startproject <project> .
$ python manage.py runserver 9000
{% endhighlight %}

# Update Installed Apps

Now need to find the `<project>/settings.py` and update `INSTALLED_APPS` with the django API rest framework.  Add `rest_framework` underneath the other django apps and before any other custom apps.

{% highlight bash %}
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'rest_framework',
    "corsheaders",
    'jwtauth',
]
{% endhighlight %}