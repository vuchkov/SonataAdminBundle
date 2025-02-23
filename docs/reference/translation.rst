Translation
===========

There are two main catalogue names in an Admin class:

* ``SonataAdminBundle``: this catalogue is used to translate shared messages
  across different Admins
* ``messages``: this catalogue is used to translate the messages for the current
  Admin

Ideally the ``messages`` catalogue should be changed to avoid any issues with
other Admin classes.

You can configure the catalogue for the Admin class by injecting the value through the container:

.. configuration-block::

    .. code-block:: xml

        <!-- config/services.xml -->

        <service id="sonata.page.admin.page" class="Sonata\PageBundle\Admin\PageAdmin">
            <tag name="sonata.admin" model_class="Application\Sonata\PageBundle\Entity\Page" manager_type="orm" group="sonata_page" label="Page"/>
            <call method="setTranslationDomain">
                <argument>SonataPageBundle</argument>
            </call>
        </service>

An Admin instance always gets the ``translator`` instance, so it can be used to
translate messages within the ``configureFields`` method or in templates.

.. code-block:: jinja

    {# the classical call by using the twig trans helper #}
    {{ 'message_create_snapshots'|trans({}, 'SonataPageBundle') }}

    {# by using the admin trans method with hardcoded catalogue #}
    {{ 'message_create_snapshots'|trans({}, 'SonataPageBundle') }}

    {# by using the admin trans with the configured catalogue #}
    {{ 'message_create_snapshots'|trans({}, admin.translationdomain) }}

The last solution is most flexible, as no catalogue parameters are hardcoded, and is the recommended one to use.

Translate field labels
----------------------

The Admin bundle comes with a customized form field template. The most notable
change from the original one is the use of the translation domain provided by
either the Admin instance or the field description to translate labels.

Overriding the translation domain
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The translation domain (message catalog) can be overridden at either the form
group or individual field level.

If a translation domain is set at the group level it will cascade down to all
fields within the group.

Overriding the translation domain is of particular use when using
:doc:`extensions`, where the extension and the translations would
be defined in one bundle, but implemented in many different Admin instances.

Setting the translation domain on an individual field::

    use Symfony\Component\Form\Extension\Core\Type\CheckboxType;

    $form
        ->with('form.my_group')
            ->add('publishable', CheckboxType::class, [], [
                'translation_domain' => 'MyTranslationDomain',
            ])
        ->end()
    ;

The following example sets the default translation domain on a form group and
over-rides that setting for one of the fields::

    use Symfony\Component\Form\Extension\Core\Type\CheckboxType;
    use Symfony\Component\Form\Extension\Core\Type\DateType;

    $form
        ->with('form.my_group', ['translation_domain' => 'MyDomain'])
            ->add('publishable', CheckboxType::class, [], [
                'translation_domain' => 'AnotherDomain',
            ])
            ->add('start_date', DateType::class, [], [])
        ->end()
    ;

Translation can also be disabled on a specific field by setting
``translation_domain`` to ``false``.

Setting the label name
^^^^^^^^^^^^^^^^^^^^^^

By default, the label is set to a sanitized version of the field name. A custom
label can be defined as the third argument of the ``add`` method::

    // src/Admin/PageAdmin.php

    final class PageAdmin extends AbstractAdmin
    {
        protected function configureFormFields(FormMapper $form): void
        {
            $form
                ->add('isValid', null, [
                    'required' => false,
                    'label' => 'label.is_valid',
                ])
            ;
        }
    }

Label strategies
^^^^^^^^^^^^^^^^

There is another option for rapid prototyping or to avoid spending too much time
adding the ``label`` key to all option fields: **Label Strategies**. By default
labels are generated by using the following rule:

    ``isValid => Is Valid``

The ``AdminBundle`` comes with different key label generation strategies:

* ``sonata.admin.label.strategy.native``: DEFAULT - Makes the string human readable
    ``isValid`` => ``Is Valid``
* ``sonata.admin.label.strategy.form_component``: The default behavior from the Form Component
    ``isValid`` => ``Isvalid``
* ``sonata.admin.label.strategy.underscore``: Changes the name into a token suitable
  for translation by prepending "form.label" to an underscored version of the field name
  ``isValid`` => ``form.label_is_valid``
* ``sonata.admin.label.strategy.noop``: does not alter the string
    ``isValid`` => ``isValid``

``sonata.admin.label.strategy.underscore`` will be better for i18n applications
and ``sonata.admin.label.strategy.native`` will be better for native (single) language
applications based on the field name. It is reasonable to start with the ``native``
strategy and then, when the application needs to be translated using generic keys, the
configuration can be switched to ``underscore``.

The strategy can be quickly configured when the Admin class is registered in
the Container:

.. configuration-block::

    .. code-block:: xml

       <!-- config/services.xml -->

        <service id="app.admin.project" class="App\Admin\ProjectAdmin">
            <tag
                name="sonata.admin"
                model_class="App\Entity\Project"
                manager_type="orm"
                group="Project"
                label="Project"
                label_translator_strategy="sonata.admin.label.strategy.native"
             />
        </service>

.. note::

    In all cases the label will be used by the ``Translator``. The strategy is
    a quick way to generate translatable keys. It all depends on the project's requirements.

.. note::

    When the strategy method is called, ``context`` (breadcrumb, datagrid, filter,
    form, list, show, etc.) and ``type`` (usually link or label) arguments are passed.
    For example, the call may look like: ``getLabel($label_key, 'breadcrumb', 'link')``
