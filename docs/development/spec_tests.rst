Serverspec Tests
================

serverspec_ tests verify the end state of the vagrant machines. Any
changes to the Ansible configuration should have a corresponding
spectest.

.. _serverspec: http://serverspec.org

Install directions (Ubuntu/Debian)
---------------------------

.. code:: sh

    apt-get install bundler
    cd spec_tests/
    bundle update

.. note:: If you run into an error regarding the version of
          ``bundler`` such as "Bundler could not find compatible versions",
          then you can install and run a particular version of ``bundler`` via:

          .. code:: sh

              gem install bundler -v 1.12.5
              bundle _1.12.5_ install
              bundle _1.12.5_ exec rake spec:development


Running the tests
-----------------

In order to run the tests, each VM will be created and provisioned, if
necessary.  Running all VMs concurrently may cause performance
problems if you have less than 8GB of RAM. You can isolate specific
machines for faster testing:

.. code:: sh

    $ cd spec_tests
    $ bundle exec rake --tasks # check output for desired machine
    rake spec:app-prod     # Run spectests against app-prod
    rake spec:app-staging  # Run spectests against app-staging
    rake spec:build        # Run spectests against build
    rake spec:development  # Run spectests against development
    rake spec:mon-prod     # Run spectests against mon-prod
    rake spec:mon-staging  # Run spectests against mon-staging

    $ bundle exec rake spec:staging

The invocation above will run tests only against the ``staging`` VMs.
You can run against multiple environments in a single invocation:

.. code:: sh

    $ cd spec_tests
    $ bundle exec rake spec:development spec:staging

Test failure against any host will cause Serverspec to exit, and stop
executing tests against subsequent hosts. In order to run the ``spec:prod``
task, you will need to configure :ref:`SSH access over Tor <ssh_over_tor>`.

Updating the tests
------------------

Changes to the ansible config should result in failing spectests, but
only if an existing task was modified. If you add a new task, make
sure to add a corresponding spectest to validate that state after a
new provisioning run. Tests import variables from separate YAML files
than the Ansible playbooks: ::

    spec_tests/spec/vars
    ├── development.yml
    └── staging.yml

Any variable changes in the Ansible config should have a corresponding
entry in these vars files. These vars are dynamically loaded for each
host via the ``spec_helper.rb`` file. Make sure to add your tests to
relevant location for the host you plan to test: ::

    spec_tests/spec/app-staging
    ├── apache_spec.rb
    ├── apparmor_spec.rb
    ├── iptables_spec.rb
    ├── ossec_agent_spec.rb
    ├── securedrop_app_spec.rb
    ├── securedrop_app_test_spec.rb
    └── tor_spec.rb

In the example above, to add a new test for the ``app-staging`` host,
add a new file to the ``spec_tests/spec/app-staging`` directory.

Spectest layout
---------------

The serverspec tests are mostly broken up according to machines in the
Vagrantfile: ::

    spec_tests/spec
    ├── app-staging
    ├── build
    ├── common-development
    ├── common-staging
    ├── development
    ├── mon-staging
    └── vars

There are a few exceptions:

-  ``common-development`` shares tests between ``development`` and
   ``app-staging``
-  ``common-staging`` shares tests between ``app-staging`` and
   ``mon-staging``

Ideally the serverspec tests would be broken up according to roles,
mirroring the Ansible configuration. Prior to the reorganization of
the Ansible layout, the tests are rather tightly coupled to hosts. The
layout of spectests is therefore subject to change.
