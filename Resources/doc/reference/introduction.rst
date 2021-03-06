Introduction
============

A Page
------

The ``PageBundle`` is a special kind of CMS as it handles different types of page.
From within a Symfony2 application, actions are used to render a HTML response. When
you need to add a new component (or block) inside an action, you need to edit the
template. In the other hand, in the CMS world, users edit area and manage
content but it is not possible to have a complex actions or workflows.

It is very hard to mix CMS page and Action page inside one and unique solution, the
easiest way is to build the project with 2 backends, one for the CMS and one for
the application.

The ``PageBundle`` try to solve this problem by encapsuling action pages into the CMS.
So actions can be handled as a CMS pages with the same solution, and this allows
to easily include externals Symfony bundles.

Page types :

    - ``CMS Page`` : a standard CMS page with url
    - ``Hybrid Page`` : a page linked to a Symfony action, this can be any kind of url
      matched by the router.
    - ``Dynamic Page`` : a dynamic page is an hybrid page with parameters
      ie, /blog/{year}/{month}/{slug}
    - ``Internal Page`` : page shared across different pages, very usefull for handling
      footer and header

A Block
-------

The ``PageBundle`` does not know how to manage contents, actually there is no content
management. This part is delegated to services. The bundle only manage references to
service required by one page. Reference informations are stored inside a ``Block``.

A block is a small unit, it contains a few information :

    - service id
    - position
    - settings used by the service

Every block service must implements the ``Sonata\PageBundle\Block\BlockServiceInterface``
which defines a set of function to create so the service can be integrated nicely with
editing workflow. The important information is that a block service must always return
a ``Response`` object.

By default the ``PageBundle`` is shipped with core block services :

    - sonata.page.block.container      : Block container
    - sonata.page.block.action         : Render a specific action
    - sonata.page.block.text           : Render a text block
    - sonata.page.block.children_pages : Render a navigation panel

A Snapshot
----------

A ``Snapshot`` is a version of a ``Page``, the snapshot is used to render the page
to the final user. So when an editor organizes ``Page`` and ``Block`` the final user
does not see these modifications unless the editor create a new snapshot of the page.

A Cache
-------

There is a cache mechanism integrated into the bundle, each block service is linked
to a cache service. The bundle provides different kind of cache handler :

    - sonata.page.cache.noop        : default, no cache
    - sonata.page.cache.esi         : Edge Side Include cache
    - sonata.page.cache.mongo       : MongoDB cache backend
    - sonata.page.cache.memcached   : Memcached cache backend
    - sonata.page.cache.apc         : Apc cache backend
    - sonata.page.cache.js_sync     : Javascript synchronized load
    - sonata.page.cache.js_async    : Javascript asynchronized load

Depends on the block logic some cache backends are more suitable than other :

 - Container should used the ``sonata.page.cache.esi`` cache
 - Users' related block like basket summary or authentication area should
   used the ``sonata.page.cache.js_async`` cache.

Of course if you don't have a reverse proxy server you can use other caching solution
such as memcached, mongo or apc.

The ``sonata.page.cache.noop`` cache can be use if you don't want caching!

