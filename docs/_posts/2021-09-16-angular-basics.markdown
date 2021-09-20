---
layout: single
title:  "Angular Basics"
toc: true
date:   2021-09-16 20:00:00 +1000
categories: angular typescript bootstrap
---

## Installation

{% highlight bash %}
$ sudo npm install -g @angular/cli
{% endhighlight %}

## Creating a new Project

Use the CLI tools included with angular to create a new project.  The ng new command will create a folder in the current directory based on the name given.

{% highlight bash %}
$ cd /path/to/create/project
$ ng new project-name
$ cd ./project-name
{% endhighlight %}

## Testing the App

Assuming in the directory of the project, test the web application using the below command.  The output will show the port that is serving the webapp on the localhost.

{% highlight bash %}
$ ng serve
{% endhighlight %}

## Creating Components

The angular CLI tools can be used to create components rather then creating manually.  Navigate to the directory where you want the component created.  The below will create a component named "ButtonComponent".

{% highlight bash %}
$ ng create component button
{% endhighlight %}

Running the above will give the below output.  

{% highlight bash %}
CREATE src/app/button/button.component.html (21 bytes)
CREATE src/app/button/button.component.spec.ts (626 bytes)
CREATE src/app/button/button.component.ts (275 bytes)
CREATE src/app/button/button.component.css (0 bytes)
UPDATE src/app/app.module.ts (396 bytes)
{% endhighlight %}

|Filename|Description|
|---|---|
|button.component.html|This contains the html to display the component|
|button.component.ts|This contains the logic behind the component|
|button.component.css|This contains the stylesheet for the component|
|button.component.spec.ts|Don't know what this does but if I need to touch it then I might upate this!|

## Using a Component

First you will need to remove the initial markup assuming starting with a new project.

1. Go to src/app/app.component.html and remove the html contained in this file.
2. While in src/app/app.component.html (and is empty) add html to display your component.  The is the value of selector in your component ts file.
{% highlight angular %}
<app-button></app-button>
{% endhighlight %}
3. Update your component html to do something.
{% highlight angular %}
<button type="button">Click Me!</button>
{% endhighlight %}
4. Run ng server and view the output in the browser.

## Templates and Directives

Templates is term given in Angular when you want to display data in your component or incorporate some sort of logic in the html output.

Directives are Angular Classes that add additional behaviour to elements.

A component named listComponent with its backend logic is shown below.  This code is the same as the output from ng generate but with the two class members added for the title and list data to display.

{% highlight typescript %}
import { Component, OnInit } from '@angular/core';

@Component({
  selector: 'app-list',
  templateUrl: './list.component.html',
  styleUrls: ['./list.component.css']
})
export class ListComponent implements OnInit {

  listTitle: string = 'A list of Colours';
  listData: string[] = ['red', 'blue', 'green'];

  constructor() { }

  ngOnInit(): void {
  }

}
{% endhighlight %}

The html for this listComponent is shown below.  The double curly braces can be used to embed a typescript expression to display data in the corresponding class.

{% highlight html %}
{% raw %}

## {{ title }}>

<ul>
    <li *ngFor="let data of listData">{{ data }}</li>
</ul>
{% endraw %}
{% endhighlight %}

The output of the above example in the browser is shown below.

![Output of the listComponent](/images/angular-basics/templates-directives.PNG "Templates and Directives output")

## Services

Services are used to interact with other systems in order to get or store data to be used in the application.

Use the Angular CLI tools to generate a service.

{% highlight bash %}
$ ng generate service colour
{% endhighlight %}

Output:
{% highlight bash %}
CREATE src/app/colour.service.spec.ts (357 bytes)
CREATE src/app/colour.service.ts (135 bytes)
{% endhighlight %}

Now instead of defining data in the app we can use the service to return the colours for the application.

{% highlight typescript %}
import { Injectable } from '@angular/core';

@Injectable({
  providedIn: 'root'
})
export class ColourService {

  getColours() {
    return ['red', 'blue', 'green'];
  }

  constructor() { }
}
{% endhighlight %}

## Dependency Injection

To use injection with a service it needs to be added to the dependency injection system.  When using the cli command to generate the service it will include the "root" provider.  To use the service in a component add the import and inject it by adding it to the constructor.

{% highlight typescript %}
import { Component, OnInit } from '@angular/core';
import { ColourService } from '../colour.service';

@Component({
  selector: 'app-list',
  templateUrl: './list.component.html',
  styleUrls: ['./list.component.css']
})
export class ListComponent implements OnInit {

  listTitle: string = 'A list of Colours';
  listData: string[];

  constructor(colourService : ColourService) {
    this.listData = colourService.getColours();
   }

  ngOnInit(): void {
  }

}

{% endhighlight %}

## Adding Bootstrap

To add bootstrap CSS run the below npm command from within the project directory.

{% highlight bash %}
$ npm install boostrap --save

{% endhighlight %}

Now add bootstrap to the global styles in styles.css

{% highlight css %}
@import "~bootstrap"

{% endhighlight %}

## Bootstrap Icons

Boostrap icons can also be used.

{% highlight bash %}
$ npm install boostrap-icons --save

{% endhighlight %}

Update `styles.css`

{% highlight css %}
@import "~bootstrap";
@import "~bootstrap-icons/font/bootstrap-icons.css";

{% endhighlight %}

Then use an icon in template

{% highlight html %}
<i class="bi-alarm" style="font-size: 3rem; color: cornflowerblue;"></i>
{% endhighlight %}

