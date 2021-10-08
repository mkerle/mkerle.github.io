---
layout: single
title:  "Django - Basics"
date:   2021-09-27 16:30:00 +1000
toc: true
categories: python django vscode
---

## Requirements

Install python and VS Code.  Install `pipenv`.

{% highlight bash %}
$ python -m pip install --upgrade pip
$ pip install pipenv
{% endhighlight %}

## Setting up a Django Virtual Environment

Create a new python virtual environment and install Django.  NOTE: For django projects, do not use "-" in the directory/project name.

{% highlight bash %}
$ cd ~/<somedir>
$ mkdir <project>
$ cd <project>
$ pipenv install django
{% endhighlight %}

## Creating Django Project

The below created a django project in the current directory.  Test the project by running python with `manage.py`.  The default port is 8000 but can be specified when using runserver.

{% highlight bash %}
$ pipenv shell
$ django-admin startproject <project> .
$ python manage.py runserver 9000
{% endhighlight %}

## Setting up VS Code

Open the folder in VS Code where the project was created.  In case you use a different version of python in the virtual env then on the system, tell vs code which python interpeter to use.

{% highlight bash %}
$ pipenv --venv
{% endhighlight %}

In VS Code open the command pallete and type python intrepreter.  Copy the path from the output of `pipenv` and append `bin/python`.

In a VS Code terminal, use the below to use the virtual environment.

{% highlight bash %}
$ source <pipenv-output-path>/bin/activate
$ python manage.py runserver
{% endhighlight %}

## Creating an App in the Project

A Django project is made up of one or more apps.  To create an app in a project:

{% highlight bash %}
$ python manage.py startapp <app-name>
{% endhighlight %}

With the app created, add it to the project in `settings.py`.  In the example below `newapp` is the name of the app created above.  The `django.contrib.sessions` can be removed.

{% highlight bash %}
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'newapp'
]
{% endhighlight %}
