..   -*- mode: rst -*-
.. vim:tw=0:ts=2:sw=2:et:spell:ft=rst

##############################################
virtualenvwrapper 🙃 with *local venv* support
##############################################

This is a fork of Doug Hellmann's `virtualenvwrapper
<https://github.com/python-virtualenvwrapper/virtualenvwrapper>`__
that supports project-local Python virtualenv directories.

This fork checks the current directory for virtualenvs first,
before falling back on the typical ``~/.virtualenvs`` directory
that *virtualenvwrapper* manages.

==============
Added Features
==============

----------------------
Local-aware ``workon``
----------------------

This fork features a more flexible ``workon`` function.

- The upstream project's ``workon`` uses ``WORKON_HOME``
  as the base for the virtualenv directories.

  - The ``WORKON_HOME`` environ defaults to ``~/.virtualenvs``.

    - E.g., calling ``workon acme`` will activate the
      ``~/.virtualenvs/acme`` virtualenv.

  - But what if you want to stash a virtualenv in a project
    directory?

    - E.g., say I have a project called ``wonka`` where I
      created the virtualenv locally, e.g.,::

        $ cd /path/to/wonka
        $ python3 -m venv .venv

      Then an easier way to enter the virtualenv would be::

        $ cd /path/to/wonka
        $ workon

      Where ``workon`` in this instance is the same
      as activating the *venv* directly, e.g.,::

        $ . .venv/bin/activate

    - This is different than the normal *virtualenvwrapper*
      behavior, where a bare ``workon`` lists all the virtualenv
      directories under ``WORKON_HOME``. This fork just assumes
      you want to activate a virtualenv for the current project.

  - In summary, this fork's ``workon`` first looks for a local
    virtualenv (in the current directory), and it activates that
    if present.

    - If there's no *venv* locally, ``workon`` defers to classic
      *virtualenvwrapper* behavior, and it'll list what's under
      ``WORKON_HOME``.

-----------------------------
Handling multiple local venvs
-----------------------------

What if your local project has more than one virtualenv?

- For example, suppose I have two ``pyproject.toml`` files for the
  project:

  - There's the conventional config, ``wonka/pyproject.toml``,
    which installs all dependencies normally.

  - But there's also a special config, say,
    ``wonka/.pyproject-editable/pyproject.toml``,
    which installs some dependencies in editable mode.

- To get started, I'll create two separate virtualenv directories::

      $ cd /path/to/wonka

      $ python3 -m venv .venv-prod
      $ . .venv-prod/bin/activate
      $ poetry install
      $ deactivate

      $ python3 -m venv .venv-dev
      $ . .venv-dev/bin/activate
      $ poetry -C .pyproject-editable/ install
      $ deactivate

- And now I'd like ``workon`` to still work without a name argument:

  - When I call ``workon``, I want it to activate either ``.venv-prod``
    or ``.venv-dev``.

  - This fork activates the local virtualenv with the most recently
    touched ``activate`` file.

  - In the example above, because ``.venv-dev`` was created second,
    ``workon`` will activate ``.venv-dev``.

    - If you want ``workon`` to pick the other virtualenv,
      simply ``touch`` it, e.g.,::

        touch .venv-prod/bin/activate

      and now a bare ``workon`` will pick ``.venv-prod``.

  - Furthermore, tab-completion should only apply to the local
    virtualenv directories, e.g.,::

      $ workon <TAB>
      .venv-dev     .venv-prod

    As opposed to showing what's under ``~/.virtualenvs``
    (or what's under ``WORKON_HOME``).

- If the local directory does not contain any virtualenv
  subdirectories, then ``WORKON_HOME`` will be probed,
  like *virtualenvwrapper* normally does.

---------------------------------
Using ``pushd`` instead of ``cd``
---------------------------------

If you'd like the ``cd`` commands to use ``pushd``, not ``cd``, set::

  VIRTUALENVWRAPPER_PUSHD=1

(or to any nonempty string).

- This applies to the three ``cd`` commands::

    cdproject
    cdsitepackages
    cdvirtualenv

-------------------
Create a virtualenv
-------------------

The classic ``mkvirtualenv`` from *virtualenvwrapper* still
works, e.g.,::

  $ mkvirtualenv -a /path/to/wonka wonka

But you might find it simpler to use the builtin ``venv``
module, which has been much improved over the years, e.g.,::

  $ cd /path/to/wonka
  $ python3 -m venv .venv/

Your choice. The difference being if you want your virtualenv
in a centrally-managed location (like ``~/.virtualenvs``) then
use ``mkvirtualenv``. But if you'd like to keep the virtualenv
within the project directory, create it with the ``venv`` module
instead.

-------------
Minor changes
-------------

- Improve error message about missing ``virtualenvwrapper``
  package, to help new users. Also when hooks fail.

- ``cdsitepackages`` now works when virtualenv is not active.

- Add support for dot-prefixed virtualenv names.

--------------
Anything else?
--------------

Nope. Really, this fork is all about a glorified ``workon`` command
which lets you mix local and centralized virtualenvs.

============
Installation
============

Please refer to the `original README document
<https://github.com/landonb/virtualenvwrapper/blob/liminal/README-python-virtualenvwrapper.txt>`__
for Installation instructions and further information.

