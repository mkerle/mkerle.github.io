---
layout: single
title:  "Angular Pipes"
date:   2021-09-19 19:00:00 +1000
toc: true
categories: angular typescript pipes
---

## Built-in Pipes

There are a number of built in pipes that can be used to transform data.  As the name suggests use `|` in an expression to transform the data.

See [Angular Pipes Guide][angular-pipes] for an intro to commonly used built-in pipes.


## Custom Pipes

To implement custom pipes you can create a custom pipe class using the Angular CLI tools.

{% highlight bash %}
$ ng generate pipe upper-first
{% endhighlight %}

{% highlight bash %}
CREATE src/app/upper-first.pipe.spec.ts (204 bytes)
CREATE src/app/upper-first.pipe.ts (225 bytes)
UPDATE src/app/app.module.ts (836 bytes)
{% endhighlight %}

An example pipe that can be used to uppercase the first letter of a string is shown below:

{% highlight typescript %}
import { Pipe, PipeTransform } from '@angular/core';

@Pipe({
  name: 'upperFirst'
})
export class UpperFirstPipe implements PipeTransform {

  transform(value: string, ...args: unknown[]): unknown {
    if (value) {
      return value[0].toUpperCase() + value.substr(1);
    } else {
      return null;
    }
  }

}
{% endhighlight %}


[angular-pipes]: https://angular.io/guide/pipes