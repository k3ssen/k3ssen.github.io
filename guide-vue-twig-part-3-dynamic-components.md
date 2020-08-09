{% raw %}
# Combine Twig and Vue: 

 [Intro](guide-vue-twig.md)
| [Part 1: Passing down a vue object](guide-vue-twig-part-1-object.md)
| [Part 2: VueStorage](guide-vue-twig-part-2-storage.md)
| **[Part 3: Dynamic components](guide-vue-twig-part-3-dynamic-components.md)**
| [Part 4: v-models in your Symfony form](guide-vue-twig-part-4-form.md)
| [Part 5: fetching dynamic components](guide-vue-twig-part-5-fetching-dynamic-components.md)

## Part 3: Dynamic components

So far, you've seen how to pass data from Twig to the Vue main instance, but there might be cases
where you don't want to touch the main instance and want to use a different instance for a particular page. 

In such case you can use dynamic components: `<component :is="someVueObject"></component>`
where `someVueObject` is just like the `vue` variable object we've already been using. 

The biggest difference is that you'll need to provide a template, but this is particularly easy in Twig:

```twig
{% extends 'base.html.twig' %}

{% block body %}
    <div>
        main vue instance value = @{ value }
    </div>
    <component :is="pageComponent"></component>
{% endblock %}

{% block template %}
    <div>
        page component value = @{ value }
    </div>
{% endblock %}

{% block script %}
    <script>
        vue = {
            data: () => ({
                value: 'a value set in the main instance',
                pageComponent: {
                    delimiters: ['@{', '}'],
                    template: `{{ block('template')|raw }}`,
                    data: () => ({
                        value: 'value set in page component',
                    }),
                }
            })
        }
    </script>
{% endblock %}
```

Twig lets you separate the contents of the template by using a block, making your code much more readable.

By using the dynamic component you can use `@{ value }` twice, but with different results.

> Note: Make sure not to use a `${` delimiter like 
>[Symfony suggests](https://symfony.com/doc/current/frontend/encore/vuejs.html#using-vue-inside-twig-templates),
>because when using ticks content within `${` will be parsed as javascript-expression, which is treated different as
> variables/expressions in vue.

{% endraw %}