# Combine Twig and Vue: dynamic components

So far, you've seen how to pass data from Twig to the Vue main instance, but there might be cases
where you've already wrapped your application in a vue-instance and want to use a different instance
for a particular page. 

In such scenario you can use dynamic components: `<component :is="someVueObject"></component>`
where `someVueObject` is just like the `vue` variable object we've already been using. 
The biggest difference is that you'll need to provide a template, but this is particularly easy with Twig:

```twig
{% extends 'base.html.twig' %}

{% block body %}
    <div>
        main vue instance value = :{ value }
    </div>
    <component :is="pageComponent"></component>
{% endblock %}

{% block template %}
    <div>
        page component value = :{ value }
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

By using the dynamic component you can use `:{ value }` twice, but with different results.

> Note: Make sure not to use a `${` delimiter like 
>[Symfony suggests](https://symfony.com/doc/current/frontend/encore/vuejs.html#using-vue-inside-twig-templates),
>because when using ticks (`) content within ${ will be parsed as javascript-expression, which is treated different as
> variables/expressions in vue.
