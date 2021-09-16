.. This README is meant for consumption by humans and pypi. Pypi can render rst files so please do not use Sphinx features.
   If you want to learn more about writing documentation, please check out: http://docs.plone.org/about/documentation_styleguide.html
   This text does not appear on pypi or github. It is a comment.

==========================
collective.metadataversion
==========================

A simple helper for conditional metadata updates

The portal catalog of Zope / Plone sites holds socalled "brains" which have
metadata attributes  containing the data to be used when the catalog is
searched and the search results are listed.

Those metadata columns -- like index values -- can be customized,
and they can evolve over time, as your customization package(s) evolve(s).
If your database is small, you'll happily reindex the whole data
whenenver you change something to your metadata (or indexes).
Or if your metadata is really cheap.

But perhaps this is not the case.

Perhaps you like to be able to reindex the most important parts first,
and the rest can simply become reindexed when changed.
And then decide there are more objects which really should be reindexed
without being changed themselves.

Perhaps you'd even like to be able to handle cases where objects
with different generations of metadata setups occur.

This simple little package doesn't do anything peculiarly magic;
it just provides a little ``metadata_version`` column which holds
the current tip number of your own metadata evolution,
and it offers a little utility which helps you to reindex only those objects
which don't feature the most current metadata setup yet.


Features
========

- provides a simple ``metadata_version`` metadata attribute, and
- a simple utility to reindex objects only if their ``metadata_version`` is
  outdated


Examples
========

This is how the `.utils.make_metadata_updater` utility function is used.

In your policy package's setuphandler module, you may have::

  from collective.metadataversion.decorator import step
  from collective.metadataversion.utils import make_metadata_updater

  ...
  @step
  def update_metadata_of_prominent_objects(context, logger):
      reindex = make_metadata_updater(context, logger, 42)
      catalog = getToolByName(context, 'portal_catalog')
      updated, skipped = 0, 0
      for brain in catalog(<my fancy query>):
          if reindex(brain):
              updated += 1
          else:
              skipped += 1
      (... logging ad libitum ...)

  @step
  def update_metadata_of_remaining_objects(context, logger):
      reindex = make_metadata_updater(context, logger, 42)
      catalog = getToolByName(context, 'portal_catalog')
      updated, skipped = 0, 0
      for brain in catalog({}):
          if reindex(brain):
              updated += 1
          else:
              skipped += 1
      (... logging ad libitum ...)

The `reindex` function returned by make_metadata_updater() will reindex every
object (given by brain) which has not been recently reindexed (with
metadata_version=42), and by default refresh a "cheap" selection of indexes.

Starting with the 2nd call to the first upgrade step, nothing will be actually
reindexed anymore (unless your <fancy query> spans some more objects now), since
everything is up-to-date;
the 2nd upgrade step will update all remaining objects (which might not need to
be updated so urgently), and skip all objects caught by the <fancy query>.


Notes
-----

- The ``@step`` decorator makes sure you have a non-None logger.

- The factory function `make_metadata_updater` takes care for updating the
  registry value, provided the given `metadata_version` value is greater than
  the old one (or forced).

  This is currently the only utility we provide for this purpose.
  However, you can adjust the registry key directly yourself, if you prefer.

- There are a few keyword-only options for customization.


Translations
============

This product has been translated into

- English
- German


Installation
============

Install collective.metadataversion by adding it to your buildout::

    [buildout]
    ...
    eggs =
        collective.metadataversion


and then running ``bin/buildout``.

After your Zope instance was restarted, you'll have the
``collective.metadataversion`` package in your extensions view
(``/prefs_install_products_form``), or in the "Quick installer";
select and activate it.

After activation, you'll have a (prefixed) ``metadata_version`` key in your
configuration registry which you can adjust according to your needs.


Contribute
==========

- Issue Tracker: https://github.com/collective/metadataversion/issues
- Source Code: https://github.com/collective/metadataversion


Support
=======

If you are having issues, please let us know;
please use the `issue tracker`_ mentioned above.


License
=======

The project is licensed under the GPLv2.

.. _`issue tracker`: https://github.com/collective/metadataversion/issues

.. vim: tw=79 cc=+1 sw=4 sts=4 si et