# Combine Twig and Vue
## Part 4 - Render your Symfony form in Vue

Rendering a form in Twig could be as easy as using `{{ form(form) }}`, but when you want to use Vue in your
form things can be more complicated.

### Basic approach
One of the simplest approach would be adding `v-model` attributes to your fields.
Then in your Twig you could add something like below:
```twig
{{ vue_data('someTextField', form.someStringField.vars.data) }}
```

While this may be simple for one field, it becomes a pain if you want this for all fields, especially if you need
to take different types into consideration. For example, an expanded ChoiceType requires
that you put v-model on the radio buttons or checkboxes, but the value should be based on the ChoiceType itself.

### Automated approach

Luckily, Symfony lets you add extensions for form types and by using the `FormType::class` as extended type you
can target all types in one go.

By creating a `FormTypeExtension` we can add `v-model` to the fields that need one and we have access to all
information we need to decide the value we need to put in the v-model. Because the `VueStorage` is a service,
we can use it in this in our extensions.

```php
<?php
declare(strict_types=1);

namespace App\Form\Extension;

use App\Vue\VueStorage;
use Symfony\Component\Form\AbstractTypeExtension;
use Symfony\Component\Form\Extension\Core\Type\FormType;
use Symfony\Component\Form\FormInterface;
use Symfony\Component\Form\FormView;
use Symfony\Component\OptionsResolver\OptionsResolver;

class FormTypeExtension extends AbstractTypeExtension
{
    /**
     * @var VueStorage
     */
    private VueStorage $vueStorage;

    public function __construct(VueStorage $vueStorage)
    {
        $this->vueStorage = $vueStorage;
    }

    public static function getExtendedTypes(): iterable
    {
        return [FormType::class];
    }

    public function configureOptions(OptionsResolver $resolver): void
    {
        // Vue might not be needed in all forms; the 'use_vue' is to make sure it can be enabled only when needed.
        $resolver->setDefaults([
            'use_vue' => false,
        ]);
    }

    public function buildView(FormView $view, FormInterface $form, array $options)
    {
        // If the root has use_vue disable, then do not add v-models.
        if (!$form->getRoot()->getConfig()->getOption('use_vue', false)) {
            return;
        }
        $compound = $view->vars['compound'] ?? false;
        // Compound forms need to v-model (instead, their children will have v-models)
        if (!$compound) {
            $vModelName = $this->getVModelName($view);
            $view->vars['attr']['v-model'] = $vModelName;
            $this->setVModelValue($vModelName, $view);
        }
    }

    protected function setVModelValue(string $vModelName, FormView $view)
    {
        // In case of choice-options the value might be set already; There's no need to repeat.
        if ($this->vueStorage->has($vModelName)) {
            return;
        }
        $hasChoices = $valueView->vars['choices'] ?? false;
        // If the current view is an option, then use the parent-view for deciding what value to use.
        $valueView = $hasChoices ? $view->parent : $view;

        $value = $valueView->vars['data']; // In most cases the data is what we need.
        if (!$value && $valueView->vars['value']) {
            $value = $valueView->vars['value'];
        }
        $multiple = $valueView->vars['multiple'] ?? false;
        if ($hasChoices && $multiple) {
            $value = $value ?: []; // In case of multiple-choice (checkboxes/multi-select), the value must be an array.
        }
        $this->vueStorage->add($vModelName, $value);
    }
    
    /**
     * Decide the v-model name, which should be the name of the view prefixed by its ancestor names, where the root
     * name will be 'form'.
     * E.g. a 'name' field of a productRow subform will be 'form.productRow.name'
     */
    protected function getVModelName(FormView $view): string
    {
        $parent = $view->parent;
        if (!$parent) {
            return 'form'; // Use 'form' instead of form root-name to make it easier to reference form-models.
        }
        $name = $view->vars['name'];
        $parentName = $this->getVModelName($parent);
        // Use the parent-model name for choice-options.
        if ($view->parent->vars['choices'] ?? false) {
            return $parentName;
        }
        return $parentName ? $parentName . '.' . $name : $name;
    }
}
```

With this extension all you need to do is set the `use_vue` option to `true` in any form you want. 
This will have a `form` object added to the vue-data.