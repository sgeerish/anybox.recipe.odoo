For contributors
================

Source and tracking
~~~~~~~~~~~~~~~~~~~
The recipes are currently both hosted in a `single Git repository
<https://github.com/anybox/anybox.recipe.odoo>`_, the master branch being
``anybox.recipe.odoo``, whereas ``anybox.recipe.openerp`` is to be found in
the ``a.r.openerp-x.y`` branches (currently only 1.9 and the legacy
1.8). This unusual structure because they still have most in common,
``a.r.odoo`` being ``a.r.openerp`` without the bits for backward compatibility
before Odoo v8.

We follow the standard GitHub workflow (issues, pull requests…).
Code contributors are systematically added to the list of
contributors at the end of the README, unless they explicitely wish
not to (what GitHub does is obviously out of our scope).

Members of the "Anybox" organization have push privileges on this repository.

Using a development version
~~~~~~~~~~~~~~~~~~~~~~~~~~~

To use a local version of the recipe, you may use the ``develop``
general buildout option::

  [buildout]
  develop = /path/to/anybox.recipe.o

To track a Git branch, we find the
`gp.vcsdevelop <https://pypi.python.org/pypi/gp.vcsdevelop>`_
extension simple and useful. Here's an example for
``anybox.recipe.openerp`` (notice the ``@`` notation for the branch)::

  [buildout]
  extensions = gp.vcsdevelop
  vcs-extend-develop = git+https://github.com/anybox/anybox.recipe.odoo@a.r.openerp-1.9#egg=a.r.openerp
  vcs-update = True

.. note:: ``gp.vcsdevelop`` leverages internally pip, and the
          ``git+https`` syntax actually `comes from pip
          <https://pip.pypa.io/en/latest/reference/pip_install.html#vcs-support>`_.

.. note::
  Actually some parts of the recipe are aware of the possible use
  of ``gp.vcsdevelop`` for python dependencies, and special care of it is
  taken in the freeze and extract features. This is known to work even
  for ``zc.buildout`` itself.

Development setup
~~~~~~~~~~~~~~~~~

We recommend "developing" the source code in a virtualenv, together
with ``bzr``. For instance::

  virtualenv recipe-env
  recipe-env/bin/pip install bzr
  git clone https://github.com/anybox/anybox.recipe.odoo
  cd anybox.recipe.odoo
  python setup.py develop

Coding style
~~~~~~~~~~~~

The recipe follows the same strong code development coding principles
as many other projects:

* Style enforcement : we follow the PEP8 guidelines
* Static analysis with `flake8 <https://pypi.python.org/pypi/flake8>`_
  (combines conveniently `pep8 <https://pypi.python.org/pypi/pep8>`_
  and `pyflakes <https://pypi.python.org/pypi/pyflakes>`_).
* Unit tests: we try and test as much as possible. It is hard to achieve a
  real 100% with a tool that calls so many external processes, but
  this is mitigated by our
  :ref:`continuous integration <continuous_integration>` practice of
  doing real Odoo installations with the latest revision of the recipe.

Launching static analysis and unit tests
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Install ``nose``, ``flake8`` and, optionally, ``coverage``::

   recipe-env/bin/pip install nose coverage flake8==2.0 \
                      pep8==1.4.6 mccabe==0.2.1 pyflakes==0.7.3


.. note:: we've had problems lately with discrepancies in ``pep8``
          versions, that's why versions of ``flake8`` and its
          dependencies are fixed above. In case of doubt, check what
          the buildbot is actually running.

Run ``flake8`` and the tests (in this example, after virtualenv activation)::

    cd anybox.recipe.odoo
    flake8 anybox && nosetests anybox --with-doctest

There is also this convenience to run the tests and output a coverage report::

    source ./test-cover


.. _integration tests:

Integration tests
~~~~~~~~~~~~~~~~~

There is a special category of tests: those that need a real Odoo
instance, built with the recipe, to run.

They are located within the ``tests_with_openerp`` subdirectory and
need to be launched with a launcher script constructed by the recipe.

For example, create a testing buildout like this::

  [buildout]
  parts = odoo
  [odoo]
  # version as you wish
  version = nightly 8.0 latest
  eggs = nose
  openerp_scripts = nosetests command-line-options=-d

Then run ``bin/buildout``, create a database and initialize it. From
the buildout directory::

  createdb test-recipe
  bin/start_odoo -d test-recipe -i base --stop-after-init

You can then run the tests::

  bin/nosetests_odoo -d test-recipe -- /path/to/recipe/branch/tests_with_openerp

Currently, these tests are all about the ``Session`` objects, used in
scripts.

.. note:: you may use a different version of the recipe to build that
          testing buildout. This is anyway what happens if you build
          with your development version, and hack some changes
          afterwards.

          Using a very different version of the recipe could give
          funky results, but you're supposed to know what you're doing
          at this point.


.. _continuous_integration:

Continuous integration
~~~~~~~~~~~~~~~~~~~~~~

Basic builds
------------

Upon each push on the main branches, Anybox' public
buildbot awakes to check the coding style, run the tests and build
this documentation. You may check the status there:

* `anybox.recipe.odoo builder
  <http://buildbot.anybox.fr/waterfall?show=anybox.recipe.odoo>`_
* `anybox.recipe.openerp builder
  <http://buildbot.anybox.fr/waterfall?show=anybox.recipe.openerp>`_

Actual runs
-----------

Furthermore, this buildbot instance runs `anybox.buildbot.openerp
<https://pypi.python.org/pypi/anybox.buildbot.openerp>`_,
a buildbot configurator for Odoo installations based on the recipe.

This is used in turn to run high-level integration tests, having the
latest bzr version of the recipe actually install several combinations
of OpenObject server and addons, and run their unit tests.

The configuration is stored in the ``buildbot`` subdirectory of the
master branch. It is made of a high level configuration file
(``MANIFEST.cfg``) and buildout configuration files. This buildbot
instance actually aggregates several such configurations.

The corresponding builders are those whose name starts with
``recipe-`` in the `builders list
<http://buildbot.anybox.fr/builders>`_.

.. note:: the `integration tests`_ mentioned above are executed in
          particular during this process, currently in the
          ``recipe-7.0-postgresql-9.2`` builder.

Some builds may appear to be broken because of tests failures been
pushed by upstream in Odoo itself or in the tested addons, but it's
easy to check whether this is due to a recipe failure or not.

.. note::

   Anybox hardware resources are limited; contributing buildslaves would
   be greatly appreciated.


