.. image:: https://img.shields.io/badge/tstn--051-lsst.io-brightgreen.svg
   :target: https://tstn-051.lsst.io
.. image:: https://github.com/lsst-tstn/tstn-051/workflows/CI/badge.svg
   :target: https://github.com/lsst-tstn/tstn-051/actions/

###################################
Managing The Observing Environment.
###################################

TSTN-051
========

The observing environment is a collection of repositories that are shared between different resources of the system. With it we also provide utilities to manage the environment (change/update branches on each of the repositories) and log actions taken by users.

This is complemented by the so called "run branch", which is a common branch used by the team for hot patches to the system.

This technote describes the details of the Observing Environment and how it is managed by our team.

**Links:**

- Publication URL: https://tstn-051.lsst.io
- Alternative editions: https://tstn-051.lsst.io/v
- GitHub repository: https://github.com/lsst-tstn/tstn-051
- Build system: https://github.com/lsst-tstn/tstn-051/actions/


Build this technical note
=========================

You can clone this repository and build the technote locally if your system has Python 3.11 or later:

.. code-block:: bash

   git clone https://github.com/lsst-tstn/tstn-051
   cd tstn-051
   make init
   make html

Repeat the ``make html`` command to rebuild the technote after making changes.
If you need to delete any intermediate files for a clean build, run ``make clean``.

The built technote is located at ``_build/html/index.html``.

Publishing changes to the web
=============================

This technote is published to https://tstn-051.lsst.io whenever you push changes to the ``main`` branch on GitHub.
When you push changes to a another branch, a preview of the technote is published to https://tstn-051.lsst.io/v.

Editing this technical note
===========================

The main content of this technote is in ``index.rst`` (a reStructuredText file).
Metadata and configuration is in the ``technote.toml`` file.
For guidance on creating content and information about specifying metadata and configuration, see the Documenteer documentation: https://documenteer.lsst.io/technotes.
