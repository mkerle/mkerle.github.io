---
layout: single
title:  "Angular Data Binding and Events"
date:   2021-09-19 07:00:00 +1000
toc: true
categories: angular typescript binding events
---

## Property Binding

There are several methods to display data in a component, mainly interpolation (where double curly braces are used) and property binding.  For property binding, square brackets are used in the html around the property name and the value is the component expression.  An example is shown below:

{% highlight typescript %}
import { Component, OnInit } from '@angular/core';

@Component({
  selector: 'app-button',
  templateUrl: './button.component.html',
  styleUrls: ['./button.component.css']
})
export class ButtonComponent implements OnInit {

  status: boolean = true;

  constructor() { }

  ngOnInit(): void {
  }

}
{% endhighlight %}

{% highlight html %}
<button type="button" [disabled]="!status" >This is a button</button>
{% endhighlight %}

## Attribute Binding

Attribute binding is used when binding to attributes that do not exist as standard html attributes.

[Attribute Reference][attribute-reference] to determine if it is a valid attribute.

The below snippet of html template will throw an error
{% highlight html %}
<table>
    <tr *ngFor="let data of listData">
        <td [colspan]="colSpan">{{ data }}</td>
        <td></td>
        <td></td>
    </tr>
</table>
{% endhighlight %}

{% raw %}
Error: src/app/list/list.component.html:9:13 - error NG8002: Can't bind to 'colspan' since it isn't a known property of 'td'.

9         <td [colspan]="colSpan">{{ data }}</td>
              ~~~~~~~~~~~~~~~~~~~
{% endraw %}

To bind to an attribute "attr" needs to be prepended to the attribute name.

{% highlight html %}
<table>
    <th>Colour</th><th>Col 2</th><th>Col 3</th>
    <tr>
        <td>Dummy Val 1</td><td>Dummy Val 2</td><td>Dummy Val 3</td>
    </tr>
    <tr *ngFor="let data of listData">        
        <td [attr.colspan]="colSpan">{{ data }}</td>
        
        <td>Last Col Value</td>
    </tr>
</table>
{% endhighlight %}

## Style Binding

Style binding is similar to property binding but instead binds CSS style attributes to an expression.

{% highlight html %}
<button type="button" [disabled]="!status" [style.backgroundColor]="status ? 'blue' : 'red'">This is a button</button>
{% endhighlight %}

## Event Binding

For event binding we use round brackets around the event name and the value is an expression.

As the DOM is a tree like structure events propagate up through the tree, use `event.stopPropagation` to stop processing of the event by parent elements in the DOM.

{% highlight html %}
<button type="button" (click)="clickHandler($event)">This is a button</button>
{% endhighlight %}

{% highlight typescript %}
import { Component, OnInit } from '@angular/core';

@Component({
  selector: 'app-button',
  templateUrl: './button.component.html',
  styleUrls: ['./button.component.css']
})
export class ButtonComponent implements OnInit {

  status: boolean = true;

  clickHandler($event: any) {
    $event.stopPropagation();
    console.log($event);
  }

  constructor() { }

  ngOnInit(): void {
  }

}
{% endhighlight %}

## Variable Binding

Extending from event binding we can bind variables declared in the template code that can be used in the component logic.  A template variable is prefixed with "#".

{% highlight html %}
<input #colour (keyup.enter)="onKeyUp(colour.value)" />
{% endhighlight %}

{% highlight typescript %}
import { Component, OnInit } from '@angular/core';

@Component({
  selector: 'app-input',
  templateUrl: './input.component.html',
  styleUrls: ['./input.component.css']
})
export class InputComponent implements OnInit {

  onKeyUp(colour: string) {
    console.log(colour);
  }

  constructor() { }

  ngOnInit(): void {
  }

}
{% endhighlight %}

## Two-Way Binding

The previous binding methods shown are for component to view and not view to component.  To get two way binding we use the banna-in-a-box syntax `[()]`.  The forms module will also need to be added to app.component.ts.

{% highlight typescript %}
// app.component.ts
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';
import { FormsModule } from '@angular/forms';

import { AppComponent } from './app.component';
import { InputComponent } from './input/input.component';

@NgModule({
  declarations: [
    AppComponent,
    InputComponent
  ],
  imports: [
    BrowserModule,
    FormsModule
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
{% endhighlight %}

{% highlight typescript %}
// input.component.ts
import { Component, OnInit } from '@angular/core';

@Component({
  selector: 'app-input',
  templateUrl: './input.component.html',
  styleUrls: ['./input.component.css']
})
export class InputComponent implements OnInit {

  colour: string = 'orange';

  onKeyUp() {
    console.log(this.colour);
  }

  constructor() { }

  ngOnInit(): void {
  }

}
{% endhighlight %}

{% highlight html %}
//input.component.html

<input [(ngModel)]="colour" (keyup.enter)="onKeyUp()" />
{% endhighlight %}

[attribute-reference]: https://developer.mozilla.org/en-US/docs/Web/HTML/Attributes

