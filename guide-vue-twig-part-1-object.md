{% raw %}
# Combine Twig and Vue

 [Intro](guide-vue-twig.md)
| **[Part 1: Passing down a vue object](guide-vue-twig-part1-object.md)**
| [Part 2: VueStorage](guide-vue-twig-part-2-storage.md)
| [Part 3: Dynamic components](guide-vue-twig-part-3-dynamic-components.md)
| [Part 4: v-models in your Symfony form](guide-vue-twig-part-4-form.md)

## Part 1: Passing down a vue object

### app.js

After installing webpack, the `/assets/js/app.js` should've been created for you. 

To enable vue, you only need something like below:
```js
// assets/js/app.js
import '../css/app.css'; // replace '.css with '.scss' if you're using sass.
import Vue from 'vue';
new Vue({
    el: '#app',
    delimiters: ['@{', '}'], // Twig already uses '{{' and '}}' delimiters, so here we specify an alternative.
})
```
This will apply vue to the element that has `id="app"` set, which you could use anywhere
inside your twig code.

We want to retrieve data from Twig. To achieve this let's accept a global `vue` object.
Also, we might not need Vue in all pages, so we may want to only initialize it when
we actually provide the global `vue` object. Let's update our code accordingly:

```js
import '../css/app.css'; // replace '.css with '.scss' if you're using sass.

// Only import & load vue if the 'vue' var is set and is an object, so that other pages have smalled js files to load.
if (typeof vue === 'object') {
    (async () => {
        const VueImport = await import('vue');
        const Vue = VueImport.default;
        new Vue(Object.assign({
            el: '#app',
            delimiters: ['@{', '}'], // Twig already uses '{{' and '}}' delimiters, so here we specify an alternative.
        }, vue));
    })();
}
```


### base.html.twig

Have your file look like below:
```twig
<!DOCTYPE html>
<html lang="en">
    <head>
        <meta charset="UTF-8">
        <title>{% block title %}Welcome!{% endblock %}</title>
        {% block stylesheets %}
            {{ encore_entry_link_tags('app') }}
        {% endblock %}
    </head>
    <body>
        <div id="app">
            {% block body %}{% endblock %}
        </div>
        {% block javascripts %}
            {% block script %}{% endblock %}
            {{ encore_entry_script_tags('app') }}
        {% endblock %}
    </body>
</html>
```
This is just a bare base example that makes sure your assets are loaded. 
Here we put the body inside an div with 'app' to make sure that the app-element
is available to all pages.
Additionally, a `script` block is added inside javascript to be used in
other twig files.

### Example dashboard page

Now let's create a page that uses twig.

Run `php bin/console make:controller Dashboard` to
create a Dashboard controller with `templates/dashboard/index.html.twig` file and
replace the content of this `index.html.twig` file with the following:
```twig
{% extends 'base.html.twig' %}

{% block body %}
    <p>
        @{ seconds } seconds have past since you've loaded this page.
    </p>
    <p v-if="seconds > 5">
        More than 5 seconds have past.
    </p>
{% endblock %}

{% block script %}
    <script>
        vue = {
            data: () => ({
                seconds: 0,
            }),
            created() {
                setInterval(() => { this.seconds++; }, 1000);
            },
        };
    </script>
{% endblock %}
```

In this file the `vue` object is created. It can contain all the logic your
vue-instance needs. You can put general stuff like delimiters in your app.js, but if
you ever need you can overwrite it in this object.

Since twig holds server-side data you can put any data here in this object, making it
available to your vue-instance.

{% endraw %}