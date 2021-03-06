You first block
===============

This quick tutorial explains how to create a RSS reader block.

A service block is just a service which must implements the ``BlockServiceInterface``
interface. There is only one instance of a service block, however there are many block
instances.

1. First namespaces
-------------------

The ``BaseBlockService`` implements some basics methods defined by the interface.
The current Rss block will extends this base class. The others `use` statements are required
by the interface and remaining methods.

.. code-block:: php

    <?php
    namespace Sonata\PageBundle\Block;

    use Symfony\Component\HttpFoundation\Response;
    use Sonata\AdminBundle\Form\FormMapper;
    use Sonata\PageBundle\Model\BlockInterface;
    use Sonata\PageBundle\Model\PageInterface;
    use Sonata\PageBundle\Block\BaseBlockService;

2. Default settings
-------------------

A block service needs settings to work properly, so to ensure consistency, the service should
define a ``getDefaultSettings`` method. In the current tutorial, the default settings are :

    - url : the feed url
    - title : the block title

.. code-block:: php

    function getDefaultSettings()
    {
        return array(
            'url'     => false,
            'title'   => 'Insert the rss title'
        );
    }

3. Form Edition
---------------

The ``PageBundle`` rely on the ``AdminBundle`` to manage form edition and keep
a good consistency.

.. code-block:: php

    public function buildEditForm(FormMapper $formMapper, BlockInterface $block)
    {
        $formMapper->addType('settings', 'sonata_type_immutable_array', array(
            'keys' => array(
                array('url', 'url', array()),
                array('title', 'text', array()),
            )
        ));
    }

The validation is done at runtime through a ``validateBlock`` method. You can call any
Symfony2 assertions, like :

.. code-block:: php

    function validateBlock(ErrorElement $errorElement, BlockInterface $block)
    {
        $errorElement
            ->with('settings.url')
                ->assertNotNull(array())
                ->assertNotBlank()
            ->end()
            ->with('settings.title')
                ->assertNotNull(array())
                ->assertNotBlank()
                ->assertMaxLength(array('limit' => 50))
            ->end();
    }

The ``sonata_type_immutable_array`` type is a specific form type which allows to edit
an array.

4. Execute
----------

The next step is the execute method, this method must return a ``Response`` object, this
object is used to render the block.

.. code-block:: php

    public function execute(BlockInterface $block, PageInterface $page, Response $response = null)
    {
        // merge settings
        $settings = array_merge($this->getDefaultSettings(), $block->getSettings());

        $feeds = false;
        if ($settings['url']) {
            $options = array(
                'http' => array(
                    'user_agent' => 'Sonata/RSS Reader',
                    'timeout' => 2,
                )
            );

            // retrieve contents with a specific stream context to avoid php errors
            $content = @file_get_contents($settings['url'], false, stream_context_create($options));

            if ($content) {
                // generate a simple xml element
                try {
                    $feeds = new \SimpleXMLElement($content);
                    $feeds = $feeds->channel->item;
                } catch(\Exception $e) {
                    // silently fail error
                }
            }
        }

        return $this->renderResponse('SonataPageBundle:Block:block_core_rss.html.twig', array(
            'feeds'     => $feeds,
            'block'     => $block,
            'settings'  => $settings
        ), $response);
    }

5. Template
-----------

A block template is very simple, in the current tutorial, we are looping on feeds or if not
defined, a error message is displayed.

.. code-block:: jinja

    {% extends 'SonataPageBundle:Block:block_base.html.twig' %}

    {% block block %}
        <h3>{{ settings.title }}</h3>

        <div class="sonata-feeds-container">
            {% for feed in feeds %}
                <div>
                    <strong><a href="{{ feed.link}}" rel="nofollow" title="{{ feed.title }}">{{ feed.title }}</a></strong>
                    <div>{{ feed.description|raw }}</div>
                </div>
            {% elsefor %}
                No feeds available.
            {% endfor %}
        </div>
    {% endblock %}

6. Service
----------

We are almost done! Now just declare the block as a service.

.. code-block:: xml

    <service id="sonata.page.block.rss" class="Sonata\PageBundle\Block\RssBlockService" public="false">
        <tag name="sonata.page.block" />
        <argument>sonata.page.block.rss</argument>
        <argument type="service" id="templating" />
    </service>

