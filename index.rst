###################################
Managing The Observing Environment.
###################################

.. abstract::

   The observing environment is a collection of repositories that are shared between different resources of the system. With it we also provide utilities to manage the environment (change/update branches on each of the repositories) and log actions taken by users.

This technote describes the details of the Observing Environment and how it is managed by our team.

Introduction
============

The distributed nature of the Vera Rubin Observatory Control System makes it challenging to manage shared application/library software packages between users and the control system.
In order to mitigate this issue we have created the "observing environment".
This is a collection of packages that are central for the observatory operation and spans component configurations, SAL Script and other user facing packages.
The idea is that both users and control system will have access to a common collection of packages that can be easily managed and audited.

Auditing capability is central to the observing environment as it allows us to identify what was being used by the control system at any given time, and also allows us to identify who made changes and when.

Here we describe how the observing environment is used a the different procedures to manage the repositories in the observing environment.

The Observing Environments at the Test Stands and the Summit
============================================================

We maintain separate observing environment installations for each instance of the control system, 
including at the Base Test Stand (BTS), Tucson Test Stand (TTS), and the Summit (production environment). 
As almost all of the repositories in the observing environment involve interactions of multiple systems, 
the Test Stands serve as a critical step to verify functionality before it is deployed on the Summit. 

Observing Environment Repositories
----------------------------------

At the time of this writing the following repositories are part of the observing environment:

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
- ts_calsys_standardscripts
- ts_ocs_calsys
- ts_wep
- ts_config_ocs
- ts_config_scheduler
- ts_config_attcs
- ts_config_mttcs

An updated list can be found in the repository `here <https://github.com/lsst-ts/ts_manage_observing_environment/blob/main/src/repos.rs>`__.

Here we provide additional details on a few of the most frequently updated repositories. 

ts_config_scheduler
^^^^^^^^^^^^^^^^^^^
The Scheduler configuration repos is one of the most frequently updated repositories in the obsenv as it contains:
- The scheduler configuration files that define the FBS surveys e.g. ``Scheduler/feature_scheduler/maintel/fbs_config_minimal_fieldsurvey.py``.
- The scheduler configuration files that define the Scheduler CSC configuration for each survey e.g. ``Scheduler/v7/maintel_fbs_lsstcam_minimal_fieldsurvey.yaml``.
- The JSON Blocks used by each scheduler e.g. ``Scheduler/observing_blocks_maintel/``. 

ts_config_ocs
^^^^^^^^^^^^^
This repository contains configuration files for some of the observatory CSCs, including the Watcher, and ESS CSCs among others.

ts_observatory_control
^^^^^^^^^^^^^^^^^^^^^^
This repository contains high-level functionality used by many of the Sal Scripts for observatory functions. 
Many SAL Script failures due to software bugs can actually be traced back down to errors in the methods defined in this repository. 

ts_ocs_calsys
^^^^^^^^^^^^^
This repository contains high-level functionality used by the Calibration Systems for both AuxTel and MainTel. 
Many calibration SAL Script failures due to software bugs can actually be traced back down to errors in the methods defined in this repository. 

ts_standardscripts, ts_externalscripts, and ts_auxtel/maintel/calsys_standardscripts
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
These repositories contain the SAL Scripts that are available for use on the Script Queue. 
When changes within the scripts are deployed to these repositories, they will be available on the next execution of the script. 
No additional operations are needed to pickup changes to the script execution code. 

When changes are applied to a script configuration yaml of an existing script, 
a user on LOVE will need to use the blue Reload Schema button for the changes to appear. 
This button will need to be clicked for each ScriptQueue. 

Developing features to repositories managed by the Observing Environment
------------------------------------------------------------------------
Most of the observing environment repositories are managed by TSSW and follow the standard development practices.
Development of new features can be initiated by anyone within the project, and each new feature should be documented and ticketed on JIRA ticket. 
New features should include unit test coverage and detailed change logs and commit messages.
Feature branches should be reviewed and merged in github, and a request to test and deploy the change should be made to the administrative team. 

Testing Changes to the Observing Environment At the Test Stands
---------------------------------------------------------------
The test stands are the primary tool for testing and validating changes to the observing environment. 
ALL changes to the observing environment (both feature and hotfix) MUST undergo testing at a test stand prior to deployment at the summit. 
The testing must be witnessed or executed by a member of the admin group responsible for deploying changes to the summit environment. 
The nature of the test will vary based on the change.
The results of the test should be recorded before the changes are marked for deployment at the summit. 
Once the test is complete, the obsenv at the test stand should be reverted to its deployment versions. 

Deploying Changes to the Summit Observing Environment
-----------------------------------------------------
For deploying changes to the summit environment, we again must separate the use-cases of standard feature deployments and emergency hot fix deployments. 
The separation between what qualifies as a hot fix deployment and a standard feature deployment is not black and white, 
but the general guidance is to use the hot fix deployments sparingly, and only to prevent time loss on-sky. 

Deploying Standard Feature Updates
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
For standard feature deployments, once the feature branch has been merged and the feature has been tested at the test stands, 
the repository with the new feature should be tagged for release.  
The deployment of the new feature tag should take place before the afternoon's daytime system checkouts. 
The time of daytime system checkouts can vary, but at the latest the deployments should be made before the afternoon tag-up meeting. 
Nominally this means changes should be deployed before 4pm Chile time.  
An update on changes made should be provided at the afternoon tag-up meeting. 

Deploying Emergency Hot Fixes
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Emergency hot fix deployments can be deployed at any time, including during operations, if needed to maintain operations. 
If needed to minimize loss of on-sky time, testing the hot fix at the base test stand may be skipped at the discretion of the obsenv admin. 
Emergency hot fixes are expected to be deployed as ticketed branches. 
Hot fix branches are only intended to be deployed for a short period of time, no longer than 72hrs on the summit. 
After this time, the hot fix should be marked for development as a new feature or it should be dropped and the obsenv reverted to the current release version.

Authorization for deploying changes to the observing Environment
----------------------------------------------------------------
A small number of administrative users has authority to make changes to the summit observing environment. 

At the time of writing, these users include:
- Tiago Ribeiro
- Michael Reuter
- Erik Dennihy
- David Sanmartim

A much larger number of users is authorized to deploy changes to the test stand observing environments. 
Users in this group are free to deploy, test, and record their test results on either of the two test stands. 
Once a new feature has been tested and verified by one of the administrative users above, 
it can be scheduled for deployment on the summit. 

The ``manage_obs_env cli`` Application
======================================

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
The most common user operations include:

- Checkout a specific branch for a particular repository:

  .. prompt:: bash 
    
    manage_obs_env --action checkout-branch --repository <repository-name> --branch-name <branch-name>

- Checkout a specific version tag for a particular repository.

  .. prompt:: bash 
    
    manage_obs_env --action checkout-version --repository <repository-name> --branch-name <version>

- Show current version of the packages:

  .. prompt:: bash

    manage_obs_env --action show-current-versions

More advanced operations 

- Reset the observing environment:

  .. prompt:: bash

    manage_obs_env --action reset

  This operation will reset the version of all repositories to their default version.
  The default version is either the version specified in the cycle build file or, if the repository has a run branch, the run branch.

The ``manage_obs_env`` has a number of other operations that are beyond the scope of this document.

Additional features of the Observing Environment
================================================

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

Systems Configured to use the Observing Environment
---------------------------------------------------

The following systems are currently setup to use the observing environment.

ScriptQueue
^^^^^^^^^^^
SAL Scripts launched by the queue will be from the observing environment repositories.

These include; ts_observatory_control, ts_standardscripts, ts_maintel_standardscripts, ts_auxtel_standardscripts, ts_calsys_standardscripts, and ts_externalscripts.
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

The Scheduler uses the observing environment for its configuration, which resides in ts_config_scheduler, and to access the scripts configuration for block validation.
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
