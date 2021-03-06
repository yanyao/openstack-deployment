`Home <index.html>`_ OpenStack-Ansible Developer Documentation

Included Scripts
================

The repository contains several helper scripts to manage gate jobs,
install base requirements, and update repository information. Execute
these scripts from the root of the repository clone. For example:

.. code:: bash

   $ scripts/<script_name>.sh

Bootstrapping
^^^^^^^^^^^^^

bootstrap-ansible.sh
--------------------

The ``bootstrap-ansible.sh`` script installs Ansible including `core`_ and
`extras`_ module repositories and Galaxy roles.

While there are several configurable environment variables which this script
uses, the following are commonly used:

* ``ANSIBLE_GIT_RELEASE`` - The version of Ansible to install.

* ``ANSIBLE_ROLE_FILE`` - The location of a yaml file which ansible-galaxy can
  consume which specifies which roles to download and install. The default
  value for this is ``ansible-role-requirements.yml``.

The script also creates the ``openstack-ansible`` wrapper tool that provides
the variable files to match ``/etc/openstack_deploy/user_*.yml`` as
arguments to ``ansible-playbook`` as a convenience.

.. core: https://github.com/ansible/ansible-modules-core
.. extras: https://github.com/ansible/ansible-modules-extras

bootstrap-aio.sh
----------------

The ``bootstrap-aio.sh`` script prepares a host for an `All-In-One`_ (AIO)
deployment for the purposes of development and gating. The script creates the
necessary partitions, directories, and configurations. The script can be
configured using environment variables - more details are provided on the
`All-In-One`_ page.

.. All-In-One: quickstart-aio

Development and Testing
^^^^^^^^^^^^^^^^^^^^^^^

run-playbooks.sh
----------------

The ``run-playbooks`` script is designed to be executed in development and
test environments and is also used for automated testing. It executes actions
which are definitely **not** suitable for production environments and must
therefore **not** be used for that purpose.

In order to scope the playbook execution there are several ``DEPLOY_``
environment variables available near the top of the script. These are used
by simply exporting an override before executing the script. For example,
to skip the execution of the Ceilometer playbook, execute:

.. code-block:: bash

    export DEPLOY_CEILOMETER='no'

The default MaxSessions setting for the OpenSSH Daemon is 10. Each Ansible
fork makes use of a Session. By default Ansible sets the number of forks to 5,
but the ``run-playbooks.sh`` script sets the number of forks used based on the
number of CPU's on the deployment host up to a maximum of 10.

If a developer wishes to increase the number of forks used when using this
script, override the FORKS environment variable. For example:

.. code-block:: bash

    export FORKS=20

run-tempest.sh
--------------

The ``run-tempest.sh`` script runs Tempest tests from the first utility
container. This is primarily used for automated gate testing, but may also be
used through manual execution.

Configurable environment variables:

* ``TEMPEST_SCRIPT_PARAMETERS`` - Defines tests to run. Values are passed to
  ``openstack_tempest_gate.sh`` script, defined in the ``os_tempest`` role.
  Defaults to ``scenario heat_api cinder_backup``.

Lint Tests
----------

Python coding conventions are tested using `PEP8`_, with the following
convention exceptions:

* F403 - 'from ansible.module_utils.basic import \*'
* H303 - No wildcard imports

Testing may be done locally by executing:

.. code-block:: bash

    tox -e pep8

Bash coding conventions are tested using `Bashate`_, with the following
convention exceptions:

* E003: Indent not multiple of 4. We prefer to use multiples of 2 instead.
* E006: Line longer than 79 columns. As many scripts are deployed as templates
        and use jinja templating, this is very difficult to achieve. It is
        still considered a preference and should be a goal to improve
        readability, within reason.
* E040: Syntax error determined using `bash -n`. As many scripts are deployed
        as templates and use use jinja templating, this will often fail. This
        test is reasonably safely ignored as the syntax error will be
        identified when executing the resulting script.

Testing may be done locally by executing:

.. code-block:: bash

    tox -e bashate

Ansible is lint tested using `ansible-lint`_.

Testing may be done locally by executing:

.. code-block:: bash

    tox -e ansible-lint

Ansible playbook syntax is tested using ansible-playbook.

Testing may be done locally by executing:

.. code-block:: bash

    tox -e ansible-syntax

A consolidated set of all lint tests may be done locally by executing:

.. code-block:: bash

    tox -e linters

.. PEP8: https://www.python.org/dev/peps/pep-0008/
.. Bashate: https://github.com/openstack-dev/bashate
.. ansible-lint: https://github.com/willthames/ansible-lint

Documentation Build
-------------------

Documentation is developed in `reStructureText`_ (RST) and compiled into
HTML using Sphinx.

Documentation may be built locally by executing:

.. code-block:: bash

    tox -e docs

.. reStructureText: http://docutils.sourceforge.net/rst.html

Gating
^^^^^^

Every commit to OpenStack-Ansible is verified by OpenStack-CI through the
following jobs:

* ``gate-openstack-ansible-docs``: This job executes the
  `Documentation Build`_.

* ``gate-openstack-ansible-linters``: This job executes the `Lint Tests`_.

* ``gate-openstack-ansible-dsvm-commit``: This job executes the
  ``gate-check-commit.sh`` script which executes a convergence test and then a
  functional test.

  The convergence test is the execution of an AIO build
  which aims to test the primary code path for a functional environment. The
  functional test then executes OpenStack's Tempest testing suite to verify
  that the environment that has deployed successfully actually works.

  While this script is primarily developed and maintained for use in
  OpenStack-CI, it can be used in other environments.

--------------

.. include:: navigation.txt
