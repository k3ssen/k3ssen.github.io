{% raw %}
# Combine Twig and Vue

 [Intro](guide-vue-twig.md)
| [Part 1: Passing down a vue object](guide-vue-twig-part-1-object.md)
| **[Part 2: VueStorage](guide-vue-twig-part-2-storage.md)**
| [Part 3: Dynamic components](guide-vue-twig-part-3-dynamic-components.md)
| [Part 4: v-models in your Symfony form](guide-vue-twig-part-4-form.md)
| [Part 5: fetching dynamic components](guide-vue-twig-part-5-fetching-dynamic-components.md)

## Part 2: VueStorage

In the previous part you've seen how you can pass data from Twig to Vue using a 
global object. 
When you're passing data quite often, you'll often need to do things like this:
```twig
{% block script %}
    <script>
        vue = {
            data: () => ({
                someObject: {{ someObject | json_encode | raw }},
                anotherObject: {{ anotherObject | json_encode | raw }},
            }),
        }
    </script>
{% endblock %}
```

We're lazy at heart so this should be simpler. 

By creating a storage-service and some Twig functions we should be able to add data
anywhere we want and have it added to the data at one point (e.g. the base.html.twig),
so that we no longer need to concern ourselves with handling this in a script-tag.

## VueStorage

This service lets you add values for specified keys to be fetched later as json:

```php
<?php
declare(strict_types=1);

namespace App\Vue;

class VueStorage
{
    protected array $vueData = [];

    public function add(string $key, $value): void
    {
        $this->vueData[$key] = $value;
    }

    public function json(): ?string
    {
        if (!$this->vueData) {
            return null;
        }
        $dataArray = [];
        // 
        foreach ($this->vueData as $key => $value) {
            $this->assignArrayByPath($dataArray, $key, $value);
        }
        return json_encode($dataArray, JSON_THROW_ON_ERROR);
    }

    /**
     * convert paths like 'main.sub.subsub' into a sub-array.
     */
    protected function assignArrayByPath(&$arr, $path, $value) {
        $keys = explode('.', str_replace(['[', ']'], ['.', ''], $path));
        foreach ($keys as $key) {
            $arr = &$arr[$key];
        }
        $arr = $value;
    }
}
```

## VueExtension

To use the `VueStorage` service in Twig, this extension adds the `vue_data` and
`get_vue_data` functions:

```php
<?php
declare(strict_types=1);

namespace App\Twig;

use App\Vue\VueStorage;
use Twig\Extension\AbstractExtension;
use Twig\TwigFunction;

class VueExtension extends AbstractExtension
{
    private VueStorage $vueDataStorage;

    public function __construct(VueStorage $vueDataStorage)
    {
        $this->vueDataStorage = $vueDataStorage;
    }

    public function getFunctions(): array
    {
        return [
            new TwigFunction('vue_data', [$this, 'addVueData']),
            new TwigFunction('get_vue_data', [$this, 'getVueData']),
        ];
    }

    public function addVueData(String $key, $value): void
    {
        $this->vueDataStorage->add($key, $value);
    }

    public function getVueData(): ?string
    {
        return $this->vueDataStorage->json();
    }
}
```


## base.html.twig

In the `template/base.html.twig` file we retrieve the data from the VueStorage service:

```twig
{% block javascripts %}
    {% block script %}{% endblock %}
    {% set vueData = get_vue_data() | raw %}
    {% if vueData %}
        <script>
            vue = Object.assign({
                data: {{ vueData }}
            }, typeof vue === 'object' ? vue : {})
        </script>
    {% endif %}
    {{ encore_entry_script_tags('app') }}
{% endblock %}
```

## Example usage

Now, simply by calling `{{ vue_data('currentPage', currentPage) }}` the server-side
`currentPage` variable is made available to be used in Vue.

```twig
{% extends 'base.html.twig' %}

{% block body %}
    {% set currentPage = app.request.get('currentPage', 1) %}
    {{ vue_data('currentPage', currentPage) }}
    <p>
        Initial value = {{ currentPage }}.
    </p>
    <form>
        <label>
            Value
            <input type="text" name="currentPage" v-model="currentPage" />
        </label>
        <button :disable="currentPage < 1">Submit</button>
        <p v-if="currentPage < 1">
            The page number must be 1 or higher.
        </p>
    </form>
{% endblock %}
```

## Globals and Observables

The examples above show how to put the server-side data into vue-instance.
You can apply the same logic to put data into a vue-variable or observable that
can be accessed in all vue-components. 

For example, you could add the following inside your `app.js`
```js
if (typeof vueStore !== 'undefined') {
    Vue.prototype.$store = Vue.observable(vueStore);
}
if (typeof vueGlobals !== 'undefined') {
    Vue.prototype.$globals = vueGlobals;
}
```

Inside your `base.html.twig` you'd need something like below:
```twig
{% block javascripts %}
    {% block script %}{% endblock %}
    <script>
        {% set vueData = get_vue_data() %}
        {% if vueData %}
            vue = Object.assign({
                data: {{ vueData | raw  }}
            }, typeof vue === 'object' ? vue : {});
        {% endif %}
        vueStore = {{ get_vue_store() | raw }};
        vueGlobals = {{ get_vue_globals() | raw }};
    </script>
    {{ encore_entry_script_tags('app') }}
{% endblock %}
```
In your `VueExtension` and `VueStorage` you'll need to add some logic for adding
vue-store and vue-globals data, which could be nearly the same as the code for the vue-data.

Finally, inside your templates like `index.html.twig` you could use something like
below:

```twig
{% extends 'base.html.twig' %}

{% block body %}
    {{ vue_store('currentPage', currentPage) }}
    currentPage in store = @{ $store.currentPage }
{% endblock
```
In this particular example it's not that different from adding information to the 
main vue-instance data. However, you can use `$store.currentPage` in other vue components without resorting
to vue-properties or emitting values.
{% endraw %}