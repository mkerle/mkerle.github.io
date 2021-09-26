---
layout: single
title:  "Angular - Routing"
date:   2021-09-26 16:30:00 +1000
toc: true
categories: angular typescript routing
---

Work in progress for Angular routing...

## Adding Routing

Import the routing module in `app.module.ts`.

{% highlight typescript %}
import { RouterModule } from '@angular/router';
{% endhighlight %}

Define the routing using an array of objects specifying the path to component mapping in the imports of `app.module.ts`.

{% highlight typescript %}
    RouterModule.forRoot([
      { path: '/', component : HomeComponent },
      { path: 'admin/admin', component : AdminComponent },
      { path: 'user/profile', component : ProfileComponent },
    ])
{% endhighlight %}


Can display the content of the route/component using the `route-outlet` component in the template.

{% highlight html %}
<div>
    <router-outlet></router-outlet>
</div>
{% endhighlight %}