---
layout: single
title:  "Angular - Directives"
date:   2021-09-22 18:00:00 +1000
toc: true
categories: angular typescript directives
---

## Structural Directives and Attribute Directives

Structural directives modify the structure of the DOM, these directives have an `*` prefix, e.g. `*ngFor`.  Attribute directives modify the attributes of DOM elements.

## ngIf

Use ngIf to selectively display elements.

{% highlight html %}
<div *ngIf="isEnabled">Enabled</div>
<div *ngIf="!isEnabled">Disabled</div>
{% endhighlight %}

An alternative to the above is to combine ngIf with ng-template.

{% highlight html %}
<div *ngIf="isEnabled; then enabledStatus else disabledStatus"></div>
<ng-template #enabledStatus>Enabled</ng-template>
<ng-template #disabledStatus>Disabled</ng-template>
{% endhighlight %}

## ngSwitch, ngSwitchCase, ngSwitchDefault

Use ngSwitch to display content based on a value of an expression.

{% highlight html %}
<div [ngSwitch]="colour">
    <div *ngSwitchCase="'green'">Green</div>
    <div *ngSwitchCase="'red'">Red</div>
    <div *ngSwitchDefault>Unknown Colour</div>
</div>
{% endhighlight %}

## ngFor and Tracking

`ngFor` was covered in [Angular Basics - Templates and Directives]({% post_url 2021-09-16-angular-basics %}).

When dealing with large amount of items there can be performance issues having to load the same data from the backend.  Performance can be improved by using the `trackBy` feature in `ngFor` as Angular will only load objects that are not known by some id.

{% highlight html %}
<!-- component html -->
{% raw %}
<ul>
    <li *ngFor="let post of posts; trackBy: trackPost">{{post.message}}</li>
</ul>
{% endraw %}
{% endhighlight %}

{% highlight typescript %}
// component ts

posts = [ {id: 1, message: 'This is post 1'}, 
        {id: 2, message: 'This is post 2'},
        {id: 3, message: 'This is post 3'}
        ];

trackPost(index, post) {
    return post ? post.id : undefined;
}
{% endhighlight %}

## ngClass

ngClass can be used to specify an object that specifies the classes to be applied to an element.  Using styles binding as described in [Angular Data Binding and Events - Style Bindings]({% post_url 2021-09-19-angular-bindings %}) we can apply styles as in the example below:

{% highlight html %}
<i [class.bi-alarm-fill]="isFavourite" [class.bi-alarm]="!isFavourite" (click)="onClick()" style="font-size: 3rem; color: cornflowerblue;"></i>>
{% endhighlight %}

The above can be simplified using `ngClass` as below:
{% highlight html %}
<i [ngClass]="{'bi-alarm-fill' : isFavourite, 'bi-alarm' : [class.bi-alarm-fill]='!isFavouriteisFavourite'}" (click)="onClick()" style="font-size: 3rem; color: cornflowerblue;"></i>>
{% endhighlight %}

## ngStyle

In a similar way to how `ngClass` is used, we can implement style bindings using an object with the `ngStyle` directive.  Below takes the previous example and sets the colour based on the `isFavourite` attribute.  Note that in some/most cases, it is better to define css to sets styles.

{% highlight html %}
<i [ngClass]="{'bi-alarm-fill' : isFavourite, 'bi-alarm' : [class.bi-alarm-fill]='!isFavouriteisFavourite'}" (click)="onClick()" [ngStyle]="{ 'color' : isFavourite ? 'cornflowerblue' : 'black' }" style="font-size: 3rem;"></i>>
{% endhighlight %}


## Custom Directives

TBA