---
layout: single
title:  "Typescript Basics"
date:   2021-09-16 18:00:00 +1000
categories: typescript
---

<h2>Getting Started</h2>
To get started with typescript you will first need Node.js.  Once you have node, typescript can be installed using npm as per below:

{% highlight bash %}
$ sudo npm install -g typescript
{% endhighlight %}

<h2>Variables and Types</h2>

Use let keyword when declaring variables.  Typescript reference for [types][typescript-type-reference]Example of declaring variables in types shown below:

{% highlight typescript %}
let boolVar : boolean = false;
let numVar : number = 1;
let floatVar : number = 0.99;
let anyVar : any;       // use with no type checking
let strVar : string = 'this is a string';
let numArray : number[] = [100, 200, 300];

console.log('This is the value of boolVar: ' + boolVar);
console.log('This is the value of numVar: ' + numVar);
console.log('This is the value of floatVar: ' + floatVar);
console.log('This is the value of strVar: ' + strVar);
console.log('This is the value of numArray: ' + numArray);

{% endhighlight %}

<h2>Interfaces</h2>

Interfaces can be used as way to name objects.  

{% highlight typescript %}
// inline object
function printPoint(p : { x : number, y : number }) {
    console.log('X: ' + p.x + ' Y: ' + p.y);
}

printPoint({x: 10, y: 20});

// interface method
type Point = {
    x: number;
    y: number;
};

function printInterfacePoint(p: Point) {
    console.log('X: ' + p.x + ' Y: ' + p.y);
}

printInterfacePoint({x: 10, y: 20});
{% endhighlight %}

<h2>Anonymous or Arrow Functions</h2>

{% highlight typescript %}
let numArray : number[] = [100, 200, 300];

numArray.forEach( (n) => {
    console.log(n);
});
{% endhighlight %}

<h2>Classes, constructors, properties, access modifiers</h2>

The example below shows an example class.  In typescript constructors cannot be overloaded.  Class attributes can be specified in the constructor using the public or private access modifiers to avoid having to declare the attributes seperately.  A "?" following the constructor parameter means it is optional.  Get and Set properties can be defined for access to class attributes.  Use the "export" keyword so that the class can be used in other modules (outside of the file it is defined in).

{% highlight typescript %}
export class LikeButton {

    _selected : boolean = false;

    constructor(private _likeCount? : number) { }

    get likeCount() { return this._likeCount; }
    set likeCount(value) { this._likeCount = value; }

    get selected() { return this._selected; }
    set selected(value) { this._selected = value; }

    click() {
        if (this._selected) {
            this._likeCount++;
        }
    }

    toString() {
        return 'Status: ' + this._selected + ' Like Count: ' + this._likeCount;
    }
}
{% endhighlight %}

<h2>Modules</h2>

To use modules/classes in other modules they need to be imported like below where the class in the previous example is imported and used.

{% highlight typescript %}

//main.ts

import { LikeButton } from "./likeButton";

let button = new LikeButton(0);

{% endhighlight %}

<h2>Compiling Typescript</h2>

To execute typescript it needs to be compiled (converted to equivalent javascript code) and then exectured using node.js.  The typescript on this page should be compiled with at least version 5 of the standard (the -t flag).

{% highlight bash %}
$ tsc -t es5 main.ts
$ node main
{% endhighlight %}

[typescript-type-reference]: https://www.typescriptlang.org/docs/handbook/2/everyday-types.html

