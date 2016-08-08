vulners.com
===========

==========  ====================
maintainer  HubbleStack / jaredhanson11
maturity    2016.7.0
platform    All
requires    SaltStack_, :doc:`HubbleStack Nova<../../../nova/README>`
source      https://github.com/HubbleStack/Nova/blob/develop/hubblestack_nova/modules/cve_scan_v2.py
==========  ====================

.. _SaltStack: https://saltstack.com

Another major component of the Nova auditing system is the on-demand CVE
scanning and reporting. This component automates the ingestion of security
advisory announcements, and compares this data to the installed packages. This
feature was inspired by FreeBSD's VuXML integration with ``pkg audit``.

This module, ``scan-v2``, uses a public vulnerability database at
https://vulners.com. 

Queries to https://vulners.com are made either directly from the minion or
served from your ``salt://`` fileserver. The defined ``ttl`` in either case
will determine the amount of time the JSON data is cached on the minion.
Example profiles for each of these are found at ``cve.scan-v2`` and
``cve.scan-v2-salt`` respectively.

Configuration
~~~~~~~~~~~~~

The required JSON files can be downloaded using the ``utils/cve_store.py`` tool
found in the Nova repository. These downloaded files can then be served using
``salt://``. 

Usage
~~~~~

.. code-block:: shell

    salt \* hubble.audit cve.scan-v2

.. code-block:: shell

    salt \* hubble.audit cve.scan-v2-salt

utils/cve_store.py
~~~~~~~~~~~~~~~~~~

This python script will query the https://vulners.com API for the required CVE
data related to the given operating system. Data is returned in a valid JSON
format, which can be served via ``salt://``.

Usage
~~~~~

The ``cve_store.py`` utility takes a space-delimited list of ``distro-version``:

.. code-block:: shell

    python ./cve_store.py centos-7 ubuntu-16.04 debian-8

The JSON files will be downloaded and stored in the current working directory
using the naming syntax: ``<distro>_<version>.json>``.

Once you've downloaded the files you need you'll need to update or create new
profiles that will use the downloaded data.

Profiles
~~~~~~~~

A ``scan-v2`` "profile" defines three things: 

 #. ``ttl`` - how long, in seconds, should we cache the CVE data. (default: 24hrs)
 #. ``url`` - an ``http://``, ``https://`` or ``salt://`` URL where the required CVE data can be found.
 #. ``control`` - (optional) limit the CVE score reported as ``Failure``.
 #. ``score`` - (optional) severity score between 1-10.

----------

**cve.scan-v2**

The default profile, which queries https://vulners.com, looks like this:

.. code-block:: yaml
   :linenos:


    cve_scan_v2:
      ttl: 86400
      url: "http://vulners.com"
      #control:
        #score: 4

.. tip:: When the url is ``vulners.com``, this module will automatically
         determine the distribution and version and query the API accordingly.

----------

**cve.scan-v2-salt**

The alternate profile, which uses the ``salt://`` fileserver, looks like this:

.. code-block:: yaml
   :linenos:


    cve_scan_v2:
      ttl: 86400
      url: salt://hubblestack_nova/centos_7.json
      #control:
        #score: 4

If you need to support multiple distributions you'll need to create a unique
"profile" for each distribution and target accordingly in the ``top.nova``.

.. tip:: When the url is NOT ``vulners.com``, this module will simply fetch the
         URI defined. No auto-detection is done.