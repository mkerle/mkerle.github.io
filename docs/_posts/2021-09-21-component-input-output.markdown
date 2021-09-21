---
layout: single
title:  "Angular - Component Input and Output Bindings + CSS"
date:   2021-09-21 19:00:00 +1000
toc: true
categories: angular typescript binding events css
---

This post covers Angular component input and output bindings.  The example also shows integration of css to change the state of a component.  Assumes usage of bootstrap-icons.

## Input and Ouput Bindings

Components can accept inputs (inilising parameters) as well as outputs (events).

The example below shows an object containing some input data to create the component and an method that will accept an output event.

{% highlight typescript %}
// app.component.ts

import { Component } from '@angular/core';
import { LikeEventArgs } from './like/like.component.ts';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css']
})
export class AppComponent {
  title = 'angularHelloWorld';

  post = { 
    isLiked : false,
    body : 'This is the body text of the post',
    likesCount : 99
  }

  onFavouriteChanged(eventArgs : LikeEventArgs) {
    console.log('Favourite Changed: ' + eventArgs);
  }
}
{% endhighlight %}

The `app.component.html` for to display the component and bind the input and outputs would look like:

{% highlight html %}
<!-- app.component.html -->

<app-like [likesCount]="post.likesCount" [isActive]="post.isLiked"></app-like>
{% endhighlight %}


The component template showing input and outputs is shown below.

{% highlight html %}
<!-- like.component.html -->

<i class="bi-heart-fill" [class.highlighted]="isActive" (click)="onClick()"></i>{{ likesCount }}
{% endhighlight %}

The component css is as per below:

{% highlight css %}
/* like.component.css */

.bi-heart-fill {
    color: #ccc;
    font-size: 3rem;
    cursor: pointer;
}

.highlighted {
    color: deeppink;
}
{% endhighlight %}

The component logic would look like:

{% highlight typescript %}
// like.component.ts

import { Component, Input, OnInit, EventEmitter } from '@angular/core';

@Component({
  selector: 'app-like',
  templateUrl: './like.component.html',
  styleUrls: ['./like.component.css']
})
export class LikeComponent implements OnInit {

  @Input('likesCount') likesCount : number = 0;
  @Input('isActive') isActive : boolean = false;

  @Output('change') change = new EventEmitter();

  onClick() {
    this.isActive = !this.isActive;
    this.likesCount += this.isActive ? 1 : -1;
    this.change.emit({ isActive : this.isActive });
  }

  constructor() { }

  ngOnInit(): void {
  }

}

export interface LikeEventArgs {
    isActive : boolean
}

{% endhighlight %}
