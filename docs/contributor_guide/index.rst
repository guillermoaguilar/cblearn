.. _contributor_guide:

=================
Contributor Guide
=================

There are multiple ways to contribute to this project.
You can report bugs in this library or propose new ideas via `Github issues`_.
This guide describes how to contribute code or documentation.

.. _Github issues: https://github.com/cblearn/cblearn/issues


.. _developer_install:

------------
Installation
------------

Contributors should not install the package from PyPI but from the Github repository
to get the latest version and to be able to manipulate the code.
First download the repository and install the project in developer mode with
developer dependencies.

.. code-block:: bash

    $ git clone git@github.com/cblearn/cblearn.git
    $ cd cblearn
    $ pip install -e.[tests,docs]

The ``-e`` option installs the package in developer mode such that changes in the code are considered directly without re-installation.

tests
    To run the unit tests, the ``pytest`` package is required, which
    can be installed by adding the ``tests`` option to the install command.

docs
    Building these docs requires the ``sphinx`` package, which can be installed by adding the `docs` option to the install command.


Now you can run the tests and build the documentation:

.. code-block:: bash

    $ python -m pytest --remote-data  # should run all tests; this can take a while.

    $ cd docs
    $ make html  # should generate docs/_build/html/index.html


The project directory contains the code directory ``cblearn/`` and the documentation ``docs/``.
In addition, the folder contains a readme, license, multiple configuration files, and an examples folder.

-------------
Changing Code
-------------

The Python code is structured in :ref:`modules`. Each module contains
a `tests` folder with unit tests.
There should be such a test for every method and function.
Use ``pytest --cov`` to run these tests and to measure the coverage; no tests should fail.
The coverage indicates the tested fraction of code and should be close to *100%*.

All Python code follows the `PEP8 Style Guide`_. The style
of all code can be checked, running ``flake8 .`` and should print no warnings.

Every class, method, and function should have a docstring describing the functionality and parameters.
Please follow the `Google Docstring Style`_.
The docstring will be added to the :ref:`api_ref` by adding the function name in ``docs/references/index.rst``.
Check the syntax of the docstring by running ``make html`` in the ``docs/`` folder.

Types should not be added to the docstring but in the code as `type hints`_.
Typechecks can be performed using ``mypy cblearn``.

.. _PEP8 Style Guide: https://www.python.org/dev/peps/pep-0008/
.. _Google Docstring Style: https://sphinxcontrib-napoleon.readthedocs.io/en/latest/example_google.html
.. _type hints: https://docs.python.org/3/library/typing.html

Remote data tests
-----------------
Tests that require remote data, for example fetching a dataset from the internet, are marked with ``@pytest.mark.remote_data``
or ``+REMOTE_DATA`` (docstring).
These tests are skipped by default but can be run by adding the ``--remote-data`` flag to ``pytest``.

Scikit-learn estimator tests
----------------------------
``scikit-learn`` provides a test suite that should ensure the compatibility of estimators.
The estimator classes that require triplet data should return
`'triplets'=True` in the ``_get_tags`` method.
Based on this tag, our test suite extends the sklearn estimator test to handle comparison-based estimators.
This modification is not unusual; sklearn internally modifies the data and skips individual tests silently based on different tags (e.g. *pairwise*).

The modifications are:

    - Monkey-patching of ``check_estimator`` function to create triplets instead of featurized data.
    - Skipping ``check_methods_subset_invariance`` and ``check_methods_sample_order_invariance``

        These tests require a 1-to-1 relationship for X -> .transform(X).
        This will never be true for our estimators (n-to-m).
        The alternative to skipping them here would be the 'non_deterministic' tag,
        which would trigger sklearn to skip these but also additional tests.


All sklearn estimator tests can be skipped with ``pytest -m "not sklearn``.

----------------------
Changing Documentation
----------------------

The documentation is contained in the `docs/` folder.
It can be built by running ``make html``.
Open ``docs/_build/html/index.html`` in a browser to view the local build of the documentation.

The documentation is structured in multiple folders and written as `reStructuredText`_.

.. _reStructuredText: https://www.sphinx-doc.org/en/master/usage/restructuredtext/index.html

-----------------------------------
Excursion: Run Github Tests Locally
-----------------------------------

Instead of running the different tests above independently, it is also possible
to run the whole testing workflow, which is used on Github, locally.

Install nektos' `act`_ and then run `act -P ubuntu-latest=nektos/act-environments-ubuntu:18.04-full`

`act` uses docker images with preinstalled software to provide almost the same test environment as Github.
If it is not yet so, you have to `install docker`_ and, optionally, make it accessible for non-root users.

.. note::
    The docker image requires about 18 GB disk space. The first start of act might take some time,
    because it downloads about 12 GB of image files.

.. _act: https://github.com/nektos/act
.. _`install docker`: https://docs-stage.docker.com/engine/install/
.. _`accessible for nonroot user`: https://docs.docker.com/engine/install/linux-postinstall/

------------------
Publish Changes
------------------

Most contributions will change files in the code or the documentation directory, as described in the
sections below. Commit your changes to a separate *git* branch (do **not** commit to ``main``).
After changing, push this branch to Github and open a pull request to the ``main`` branch there.
Once the request is opened, automated tests are run.
If these tests indicate a problem, you can fix this problem on your branch and push again.
Once the automated tests are successful, maintainers of ``cblearn`` will review the changes and provide feedback.
Usually, after some iterations, your changes will be merged into the ``main`` branch.

.. Note:

    If you state a pull request, your changes will be published under `this open source license`_.

.. _this open source license: https://github.com/cblearn/cblearn/blob/main/LICENSE


---------------------
Release a New Version
---------------------

Versions are semantic and follow PIP440_: the version indicates ``major.minor.fix``.
Breaking changes are only allowed with major version steps.

The version number is not stored anywhere in the repository.
It is derived from the git tag by *versioneer*, so creating the tag is the version bump.
The examples below release the version ``0.4.0``; replace it with the version you release.

.. warning::

    Uploads to PyPI are immutable. A published version cannot be replaced or
    uploaded again, not even after deleting it. A mistake has to be corrected
    by releasing a new fix version.

1. **Prepare the release in a pull request.** Add a section for the new version to
   ``CHANGELOG.md``, and update the metadata in ``setup.cfg`` if the maintainers or
   the supported Python versions changed.

2. **Merge the pull request and wait for the tests on** ``main``. The release workflow
   builds and uploads, but it does not run the test suite. The tests in ``test.yml``
   are the gate.

   .. note::

       Directly after the merge, a workflow run named "Release" appears and succeeds.
       It did not publish anything. That workflow runs on every push to ``main``,
       but its upload steps only run for tags.

3. **Create the tag.** The tag is the version itself, without a ``v`` prefix, because
   ``tag_prefix`` is empty in the *versioneer* configuration:

   .. code-block:: bash

       git switch main
       git pull
       git tag 0.4.0
       git push origin 0.4.0

   Alternatively, Github can create the tag in the next step.

4. **Publish a Github release** for the tag at
   https://github.com/cblearn/cblearn/releases/new. Use the version as the title
   and the new ``CHANGELOG.md`` section as the description.

   .. note::

       Publishing the release is what triggers the deployment. Pushing the tag alone
       does not, because the workflow reacts to published releases and to pushes on
       ``main``, but not to pushed tags.

5. **Watch the deployment** in the Actions tab of the repository. The workflow uploads
   to TestPyPI first and to PyPI afterwards.

6. **Verify the published package** in a fresh environment:

   .. code-block:: bash

       python -m venv cbtest
       cbtest/bin/pip install cblearn==0.4.0
       cbtest/bin/python -c "import cblearn; print(cblearn.__version__)"

   The printed version has to be exactly the tag. A ``+`` suffix or ``0+unknown``
   means that *versioneer* did not find the tag, and the upload has to be corrected
   by releasing a new fix version.

.. _PIP440: https://peps.python.org/pep-0440/
