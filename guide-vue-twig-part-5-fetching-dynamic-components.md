{% raw %}
# Combine Twig and Vue

 [Intro](guide-vue-twig.md)
| [Part 1: Passing down a vue object](guide-vue-twig-part-1-object.md)
| [Part 2: VueStorage](guide-vue-twig-part-2-storage.md)
| [Part 3: Dynamic components](guide-vue-twig-part-3-dynamic-components.md)
| [Part 4: v-models in your Symfony form](guide-vue-twig-part-4-form.md)
| **[Part 5: fetching dynamic components](guide-vue-twig-part-5-fetching-dynamic-components.md)**

## Part 5: fetching dynamic components

So far we've been dealing loading the entire page, but when we only want to load a small part inside
a page, then re-loading an entire page might be wasteful and/or breaking user-experience.
For example, in a crud table where a dialog should be opened to edit an item you only want to load
the contents for the dialog without reloading everything else. 

Here we can utilize dynamic components, but instead of passing this vue object inside the data of
the main vue instance, we'll fetch the contents we need to create this object. This involves a
few steps:
1. Provide an url we want to fetch to put into the dynamic component.
2. Fetch the contents of the provided url.
3. Parse the content into a DOM to have style and script executed separately.
4. Strip the body of style and script and pass the remaining content into the template of the vue-object.
5. Load this vue-object as dynamic vue component.



### FetchComponent

Since this is more complex than simply passing some data it's better not to put this logic
inside Twig-code directly, but to create a vue component instead. 

For example, the code below shows the content of `/assets/js/components/FetchComponent.vue`:

```vue
<template>
    <div>
        <component v-if="component" :is="component"></component>
        <div ref="scriptContainer" style="display: none"></div>
    </div>
</template>

<script>
    export default {
        data: () => ({
            component: null,
            loading: true,
        }),
        props: {
            url: { type: String },
            vueObjectName: { type: String, default: "vue" },
        },
        async mounted() {
            await this.load();
        },
        watch: {
            async url() {
                this.$refs['scriptContainer'].innerText = ''; // Cleanup script and style container
                await this.load();
            },
        },
        methods: {
            async load() {
                // reset the vueObject to make sure we won't use an earlier object defined elsewhere.
                window[this.vueObjectName] = {};
                const response = await fetch(this.url, {headers: {'fetch': 'get'}});
                if (!response.ok) {
                    console.error('Error occurred while fetching data from ' + this.url)
                } else {
                    this.processResponseText(await response.text());
                }
            },
            processResponseText(responseText) {
                const contentElement = document.createElement('div');
                contentElement.innerHTML = responseText; // Add responseText as innerHTML, so its content can be queried.
                // script and style cannot be used inside a vue template, so handle them here
                contentElement.querySelectorAll('script, style').forEach(element => {
                    const newElement = document.createElement(element.tagName);
                    newElement.textContent = element.textContent;
                    this.$refs['scriptContainer'].append(newElement); // script/style is executed once appended to the DOM.
                    element.parentNode.removeChild(element); // remove script/style from contentElement
                });
                const vueObject = window[this.vueObjectName];
                vueObject.template = '<div>' + contentElement.innerHTML + '</div>'; // Wrap the innerHTML inside a div to make sure there's only one root element.
                this.component = vueObject;
            },
        }
    }
</script>
```

The next step is to make this FetchComponent globally available in your `app.js`:

```js
//.. other imports/code and stuff
Vue.component('FetchComponent', () => import('./components/FetchComponent'));
new Vue(Object.assign({
    el: '#app',
    delimiters: ['@{', '}'], // Twig already uses '{{' and '}}' delimiters, so here we specify an alternative.
}, vue));
```

### Usage

In Twig you can now fetch components with mininum code:
```twig
{% extends 'base.html.twig' %}
       
{% block body %}
    {{ vue_add('itemUrl', null)) }}
    <button type="button" @click="itemUrl = '{{ path('show_item', {id: 1})) }}'">Show Item 1</button>
    <button type="button" @click="itemUrl = '{{ path('show_item', {id: 2})) }}'">Show Item 2</button>
    <fetch-component v-if="itemUrl" :url="itemUrl"></fetch-component>
{% endblock %}
```

Finally, you need to make sure the content fetched by `show_item` only returns the content
to use in you dynamic content, so you want to avoid loading the entire page (after all, that's what this is all about).
For example, the template used by your showItem action could look like below:

```twig
<h1>{{ item.name }} </h1>
<p>{{ item.text }} </p>
<p>
    @{ seconds } seconds have past since you've loaded this item.
</p>
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
```

### Multi-purpose templates

The FetchComponents adds `{'fetch': 'get'}` in the headers, so you could use
something like  `app.request.headers.get('fetch')` to check if you dealing with a fetch request. 
This enables you to decide what template you want to extend:
```twig
{% set isFetchRequest = app.request.headers.get('fetch') %}
{% extends isFetchRequest ? 'fetch-base.html.twig' : 'base.html.twig' %}
```
When your `base.html.twig` is extended, the entire page will be loaded, while by extending
 `fetch-base.html.twig` you only load a body and script needed for a fetched dynamic vue component.
 
This will allow you to use a single route for loading the full webpage or just a part of it, based
on whether it's a fetch request or not.
 
 
{% endraw %}