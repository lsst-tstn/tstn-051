###################################
Managing The Observing Environment.
###################################

.. abstract::

   The observing environment is a collection of repositories that are shared between different resources of the system. With it we also provide utilities to manage the environment (change/update branches on each of the repositories) and log actions taken by users.

This is complemented by the so called "run branch", which is a common branch used by the team for hot patches to the system.

This technote describes the details of the Observing Environment and how it is managed by our team.

Introduction
============

The distributed nature of the Vera Rubin Observatory Control System makes it challenging to manage shared application/library software packages between users and the control system.
In order to mitigate this issue we have created to so called "observing environment".
This is a collection of packages that are central for the observatory operation and spans component configurations, SAL Script and other user facing packages.
The idea is that both users and control system will have access to a common collection of packages that can be easily managed and audited.

Auditing capability is central to the observing environment as it allows us to identify what was being used by the control system at any given time, and also allows us to identify who made changes and when.

In addition to the collection of packages and utilities to help manage the versions of the packages that comprises the observing environment, we have also instituted a procedure to allow concurrent contributions from multiple users to the system.
This is done by a common branch (referred to as the "run branch") that users can easily identify and contribute to.

Here we describe the procedures adopted by the run branch as well as how to manage their deployment.

The Run Branch
==============

The run branch is setup as a sprint workflow to allow continuous development with regular evaluation points to determine which changes should be merged or discarded.

For every sprint (with a standard duration of 2 weeks), the observatory support team will create a ticket associated with the observatory operations support.
The ticket number is used to define the associated run branch following the project guideline, i.e., ``tickets/<ticket-number>``.
Branches are created on an as-needed basis on the repositories that comprise the observing environment.
It is not recommended to use the run branch for repositories outside the observing environment.
However, it is permitted in cases where hot patches are required.

At the end of every sprint a member of the support team will curate the contributions from the run branch of each repository and either open pull requests to merge these contributions or work with the author of the contributions to get them merged.
Not all contributions made to the run branch will be merged back during this process.
Depending on what they are, the support team might request the author to create a dedicated ticket to host larger contributions or request that they be discarded.

The following contributions are accepted as part of the run branch for any individual repository:

- Critical hot fixes.
  These are the most common usage of the run branch, and it is intended to allow patching things quickly at the production environment.
- Minor updates to configuration.
  This are most common on the Scheduler configuration in ``ts_config_ocs`` when an update to an existing survey configuration or block is needed.
- Minor new features.
  In general new features should be avoided in the run branch.
  However, minor new features are accepted in critical situations.
  For example, a user may need a new configuration parameter on a SAL Script that exposes an existing functionality of the system that was initialy not exposed.

Contributions that fall in the category above can be pushed directly to the run branch and rolled out to the production environment.
They may also be merged back at the close out of the run.

New functionality should not be developed on the run branch
Instead, they should be developed as part of their own ticket and merged, at which point the author can coordinate with the support team to either rebase the run branch or cherry-pick the contributions.

Examples of new functionality that should *NOT* be merged with the run branch includes:

- New Scheduler configuration.
- New observing blocks.
- New SAL Scripts.
- New Observatory Control class or method in an existing class.

Managing the Observing Environment
==================================

Managing the observing environment is done using the ``manage_obs_env`` cli application.
The most common use of this application is to checkout the run branch for a selected repository in the observing environment.
This is done by ssh into one of the bastion hosts on the environment one wants to update;

- For Summit (production environment), this is azar02.cp.lsst.org.
- For Base Test Stand (BTS), this is tel-hw1.ls.lsst.org.
- For Tucson Test Stand (TTS), this is tel-hw1.tu.lsst.org.

Once logged in to the machine you will have to change to the obsenv user with:

.. prompt:: bash

   sudo -iu obsenv

Then, you can run ``manage_obs_env`` to perform operations on the observing environment.

This cli supports a number of operations to manage the version of the repositories that are part of the observing environment.
The most common user operations revolve around the run branch and includes:

- Checkout the run branch for a particular repository:

  .. prompt:: bash

    manage_obs_env --action checkout-run-branch --repository <repository-name>

- List run branch:

  .. prompt:: bash

    manage_obs_env --action list-run-branch

  This will print to stdout the name of the run branch.

- Checkout a specific branch for a particular repository:

  .. prompt:: bash 
    
    manage_obs_env --action checkout-branch --repository <repository-name> --branch-name <branch-name>

- Checkout a specific version for a particular repository.

  .. prompt:: bash 
    
    manage_obs_env --action checkout-version --repository <repository-name> --branch-name <version>

- Show current version of the packages:

  .. prompt:: bash

    manage_obs_env --action show-current-versions

More advanced operations 

- Register a new run branch:
  
  .. prompt:: bash

    manage_obs_env --action register-run-branch --branch-name <branch-name>

  This operation is usually only made once at the start of the run and only need to be repeated when a new run branch is created.

- Reset the observing environment:

  .. prompt:: bash

    manage_obs_env --action reset

  This operation will reset the version of all repositories to their default version.
  The default version is either the version specified in the cycle build file or, if the repository has a run branch, the run branch.

The ``manage_obs_env`` has a number of other operations that are beyong the scope of this document.

Web Interface
-------------

For users that prefer graphical interface, there is a web interface available to manage the observing environment.
These can be accessed at the following urls:

- Summit: https://summit-lsp.lsst.codes/obsenv-management/dashboard
- BTS: https://base-lsp.lsst.codes/obsenv-management/dashboard

.. _traceability:

Traceability
------------

One of the key features of the ``manage_obs_env`` application is that it allows us to trace the user actions and the state of the observing environment.
Whenever a user runs the ``manage_obs_env`` it will publish the action performed, alongside its parameters, time and the user name of who performed it.
It also publishes a summary of the repositories version after the action is performed.

The information is readily available in sasquatch (:sqr:`068`) under the ``obsenv`` database, and can be inspected using Chronograf.

Observing Environment Repositories
----------------------------------

At the time of this writting the following repositories are part of the observing environment:

- ts_observatory_control
- atmospec
- spectractor
- summit_extras
- summit_utils
- ts_externalscripts
- ts_observing_utilities
- ts_standardscripts
- ts_auxtel_standardscripts
- ts_maintel_standardscripts
- ts_wep
- ts_config_ocs
- ts_config_attcs
- ts_config_mttcs

An updated list can be found in the repository `here <https://github.com/lsst-ts/ts_manage_observing_environment/blob/main/src/repos.rs>`__.

Systems Configured to use the Observing Environment
---------------------------------------------------

The following systems are currently setup to use the observing environment.

ScriptQueue
^^^^^^^^^^^
SAL Scripts launched by the queue will be from the observing environment repositories.

These include; ts_observatory_control, ts_standardscripts, ts_maintel_standardscripts, ts_auxtel_standardscripts and ts_externalscripts.
Having these as part of the observing environment allows us to push updates and hot fixes on the fly.
Once a package is updated, any SAL Script launched after the update will use the updated version.
There is no need to cycle the ScriptQueue state.
However, if there is a change in SAL Script configuration schema or the change is for a new SAL Script, users might have to reload the schema or command to queue to publish the schema.

Reloading can be done directly from LOVE launch script pop up view.

.. image:: /_static/love_reload_schema.png
   :target: ../_images/love_reload_schema.png
   :alt: Reload script configuration schema from LOVE launch script pop up view.

For new SAL Scripts, use the ``run_command`` script to send the ``showSchema`` command to the ScriptQueue so it will publish the new script configuration.
The configuration for the ``run_command`` script will be something like this:

.. code-block:: yaml

  component: ScriptQueue:1 # 1 for MTQueue, 2 for ATQueue and so on.
  cmd: showSchema
  parameters:
    isStandard: true/false
    path: <path-to-the-new-script>

Nublado
^^^^^^^

For :external+obs_ops:ref:`user's Jupyter notebook <Observing-Interface-Setup-Common-Observing-Environment-in-Nublado>` on nublado to have access to the same versions of the control system packages as those used by the ScriptQueue.

Scheduler
^^^^^^^^^

The Scheduler uses the observing environment for its configuration, which resides in ts_config_ocs, and to access the scripts configuration for block validation.
This allows us to quickly update the Scheduler configuration and also ensures it has access to the same version for the scripts as the ScriptQueue to perform block configuration validation.

When the Scheduler configuration or block configuration are updated, the CSC needs to be recycled (sent to Standby and back to Enabled) in order for the new settings to be loaded.

Why is it not used for all applications?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Currently, configuring a system to use the observing environment relies on mounting a shared NFS drive.
This has shown to be unreliable/problematic, especially for system deployed on k8s which discourages us to add more systems.

The Sidecar Application
-----------------------

In order to allow systems to be configured to use the observing environment without the need of a shared NFS drive, we are currently developing a sidecar application.
This application will be comprised of a daemon process running in the background.
When started the application sets up the observing environment (an action that is part of the ``manage_obs_env`` application) and then subscribes to the actions stream :ref:`mentioned above <traceability>`.

Whenever a user performs an operation in the observing environment that is logged, the sidecar application will replicate it in its local environment.

The idea is to have this as an option to systems deployed on k8s, running them as a separate sidecar, sharing a volume with the main application.

It is also possible to use the sidecar for applications not running on k8s by setting it up in the local node.
