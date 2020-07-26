{% raw %}
# Combine Twig and Vue

 **[Intro](guide-vue-twig.md)**
| [Part 1: Passing down a vue object](guide-vue-twig-part-1-object.md)
| [Part 2: VueStorage](guide-vue-twig-part-2-storage.md)
| [Part 3: Dynamic components](guide-vue-twig-part-3-dynamic-components.md)
| [Part 4: v-models in your Symfony form](guide-vue-twig-part-4-form.md)

## Intro

Both Symfony and Vue are great, but combining them can be a hassle, especially when
you need to pass data from Twig to Vue. 

This guide shows how you can combine Twig and Vue in a flexible way:

1. **Passing down a vue object**  
A Vue instances requires just a javascript object, which can be created in Twig.
2. **VueDataStore**  
In a service you can add data you want to pass down to your Vue instance. 
3. **Dynamic components**  
For edge cases and/or more flexibility you can utilize Dynamic components.
4. **v-models in your Symfony form**  
Automatically apply v-models to your Symfony forms.

It is assumed that you have some experience with both Symfony and Vuejs.

#### Project setup

First things first: you need a symfony project with webpack installed and vue enabled:

1.  Add symfony project: ` symfony new my_project_name --full`  
(https://symfony.com/doc/current/setup.html)
2. Install encore: `composer require symfony/webpack-encore-bundle`
(https://symfony.com/doc/current/frontend/encore/installation.html)
3. enable vue in `webpack.config.js`:
```js
    // ...
    .enableVueLoader(() => {}, {
        useJsx: true
    })
```
(https://symfony.com/doc/current/frontend/encore/vuejs.html)

{% endraw %}