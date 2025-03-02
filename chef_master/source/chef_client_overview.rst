=====================================================
Chef Infra Client Overview
=====================================================
`[edit on GitHub] <https://github.com/chef/chef-web-docs/blob/master/chef_master/source/chef_client_overview.rst>`__

.. note:: For the ``chef-client`` command line tool, see `chef-client(executable) </ctl_chef_client.html>`__
   The Chef Infra Client executable can be run as a daemon.

.. list-table::
   :widths: 100 420

   * - .. image:: ../../images/icon_chef_client.svg
          :width: 100px
          :align: center

     - .. tag chef_client_summary

       Chef Infra Client is an agent that runs locally on every node that is under management by Chef Infra Server. When a Chef Infra Client is run, it will perform all of the steps that are required to bring the node into the expected state, including:

       * Registering and authenticating the node with Chef Infra Server
       * Building the node object
       * Synchronizing cookbooks
       * Compiling the resource collection by loading each of the required cookbooks, including recipes, attributes, and all other dependencies
       * Taking the appropriate and required actions to configure the node
       * Looking for exceptions and notifications, handling each as required

       .. end_tag

       .. tag security_key_pairs_chef_client

       RSA public key-pairs are used to authenticate Chef Infra Client with the Chef Infra Server every time a Chef Infra Client needs access to data that is stored on the Chef Infra Server. This prevents any node from accessing data that it shouldn't and it ensures that only nodes that are properly registered with the Chef Infra Server can be managed.

       .. end_tag

.. _the-chef-client-run:

The Chef Infra Client Run
=====================================================
.. tag chef_client_run

.. THIS TOPIC IS TRUE FOR AN UPCOMING VERSION OF CHEF INFRA CLIENT; THE BEHAVIOR OF "SYNCHRONIZE COOKBOOKS" HAS CHANGED SLIGHTLY OVER TIME AND HAS BEEN VERSIONED.

A "Chef Infra Client run" is the term used to describe the steps Chef Infra Client takes to configure a node during a run. The following diagram shows the various stages that occur during a Chef Infra Client run.

.. image:: ../../images/chef_run.png

During every Chef Infra Client run, the following happens:

.. list-table::
   :widths: 150 450
   :header-rows: 1

   * - Stages
     - Description
   * - **Get configuration data**
     - Chef Infra Client gets process configuration data from the client.rb file on the node, and then gets node configuration data from Ohai. One important piece of configuration data is the name of the node, which is found in the ``node_name`` attribute in the client.rb file or is provided by Ohai. If Ohai provides the name of a node, it is typically the FQDN for the node, which is always unique within an organization.
   * - **Authenticate to the Chef Infra Server**
     - Chef Infra Client authenticates to the Chef Infra Server using an RSA private key and the Chef Infra Server API. The name of the node is required as part of the authentication process to the Chef Infra Server. If this is the first Chef Infra Client run for a node, the chef-validator will be used to generate the RSA private key.
   * - **Get, rebuild the node object**
     - Chef Infra Client pulls down the node object from the Chef Infra Server. If this is the first Chef Infra Client run for the node, there will not be a node object to pull down from the Chef Infra Server. After the node object is pulled down from the Chef Infra Server, Chef Infra Client rebuilds the node object. If this is the first Chef Infra Client run for the node, the rebuilt node object will contain only the default run-list. For any subsequent Chef Infra Client run, the rebuilt node object will also contain the run-list from the previous Chef Infra Client run.
   * - **Expand the run-list**
     - Chef Infra Client expands the run-list from the rebuilt node object, compiling a full and complete list of roles and recipes that will be applied to the node, placing the roles and recipes in the same exact order they will be applied. (The run-list is stored in each node object's JSON file, grouped under ``run_list``.)
   * - **Synchronize cookbooks**
     - Chef Infra Client asks the Chef Infra Server for a list of all cookbook files (including recipes, templates, resources, providers, attributes, libraries, and definitions) that will be required to do every action identified in the run-list for the rebuilt node object. The Chef Infra Server provides to Chef Infra Client a list of all of those files. Chef Infra Client compares this list to the cookbook files cached on the node (from previous Chef Infra Client runs), and then downloads a copy of every file that has changed since the previous Chef Infra Client run, along with any new files.
   * - **Reset node attributes**
     - All attributes in the rebuilt node object are reset. All attributes from attribute files, environments, roles, and Ohai are loaded. Attributes that are defined in attribute files are first loaded according to cookbook order. For each cookbook, attributes in the ``default.rb`` file are loaded first, and then additional attribute files (if present) are loaded in lexical sort order. If attribute files are found within any cookbooks that are listed as dependencies in the ``metadata.rb`` file, these are loaded as well. All attributes in the rebuilt node object are updated with the attribute data according to attribute precedence. When all of the attributes are updated, the rebuilt node object is complete.
   * - **Compile the resource collection**
     - Chef Infra Client identifies each resource in the node object and builds the resource collection. Libraries are loaded first to ensure that all language extensions and Ruby classes are available to all resources. Next, attributes are loaded, followed by custom resources, and then all definitions (to ensure that any pseudo-resources within definitions are available). Finally, all recipes are loaded in the order specified by the expanded run-list. This is also referred to as the "compile phase".
   * - **Converge the node**
     - Chef Infra Client configures the system based on the information that has been collected. Each resource is executed in the order identified by the run-list, and then by the order in which each resource is listed in each recipe. Each resource in the resource collection is mapped to a provider. The provider examines the node, and then does the steps necessary to complete the action. And then the next resource is processed. Each action configures a specific part of the system. This process is also referred to as convergence. This is also referred to as the "execution phase".
   * - **Update the node object, process exception and report handlers**
     - When all of the actions identified by resources in the resource collection have been done and Chef Infra Client finishes successfully, then Chef Infra Client updates the node object on the Chef Infra Server with the node object built during a Chef Infra Client run. (This node object will be pulled down by Chef Infra Client during the next Chef Infra Client run.) This makes the node object (and the data in the node object) available for search.

       Chef Infra Client always checks the resource collection for the presence of exception and report handlers. If any are present, each one is processed appropriately.
   * - **Stop, wait for the next run**
     - When everything is configured and the Chef Infra Client run is complete, Chef Infra Client stops and waits until the next time it is asked to run.

.. end_tag

About why-run Mode
=====================================================

why-run mode is a way to see what Chef Infra Client would have configured, had an actual Chef Infra Client run occurred. This approach is similar to the concept of "no-operation" (or "no-op"): decide what should be done, but then don't actually do anything until it's done right. This approach to configuration management can help identify where complexity exists in the system, where inter-dependencies may be located, and to verify that everything will be configured in the desired manner.

When why-run mode is enabled, a Chef Infra Client run will occur that does everything up to the point at which configuration would normally occur. This includes getting the configuration data, authenticating to the Chef Infra Server, rebuilding the node object, expanding the run-list, getting the necessary cookbook files, resetting node attributes, identifying the resources, and building the resource collection and does not include mapping each resource to a provider or configuring any part of the system.

.. note:: why-run mode is not a replacement for running cookbooks in a test environment that mirrors the production environment. Chef uses why-run mode to learn more about what is going on, but also Kitchen on developer systems, along with an internal OpenStack cloud and external cloud providers to test more thoroughly.

When Chef Infra Client is run in why-run mode, certain assumptions are made:

* If the **service** resource cannot find the appropriate command to verify the status of a service, why-run mode will assume that the command would have been installed by a previous resource and that the service would not be running
* For ``not_if`` and ``only_if`` attribute, why-run mode will assume these are commands or blocks that are safe to run. These conditions are not designed to be used to change the state of the system, but rather to help facilitate idempotency for the resource itself. That said, it may be possible that these attributes are being used in a way that modifies the system state
* The closer the current state of the system is to the desired state, the more useful why-run mode will be. For example, if a full run-list is run against a fresh system, that run-list may not be completely correct on the first try, but also that run-list will produce more output than a smaller run-list

For example, the **service** resource can be used to start a service. If the action is ``:start`` and the service is not running, then start the service (if it is not running) and do nothing (if it is running). What about a service that is installed from a package? Chef Infra Client cannot check to see if the service is running until after the package is installed. A simple question that why-run mode answers is what Chef Infra Client would say about the state of the service after installing the package, because service actions often trigger notifications to other resources. So it can be important to know in advance that any notifications are being triggered correctly.

For a detailed explanation of the dry-run concept and how it relates to the why-run mode, see `this blog post <http://blog.afistfulofservers.net/post/2012/12/21/promises-lies-and-dryrun-mode/>`_.


Authentication
-----------------------------------------------------
.. tag chef_auth

All communication with the Chef Infra Server must be authenticated using the Chef Infra Server API, which is a REST API that allows requests to be made to the Chef Infra Server. Only authenticated requests will be authorized. Most of the time, and especially when using knife, Chef Infra Client, or the Chef Infra Server web interface, the use of the Chef Infra Server API is transparent. In some cases, the use of the Chef Infra Server API requires more detail, such as when making the request in Ruby code, with a knife plugin, or when using cURL.

.. end_tag

.. tag chef_auth_authentication

The authentication process ensures the Chef Infra Server responds only to requests made by trusted users. Public key encryption is used by the Chef Infra Server. When a node and/or a workstation is configured to run Chef Infra Client, both public and private keys are created. The public key is stored on the Chef Infra Server, while the private key is returned to the user for safe keeping. (The private key is a .pem file located in the ``.chef`` directory or in ``/etc/chef``.)

Both Chef Infra Client and knife use the Chef Infra Server API when communicating with the Chef Infra Server. The chef-validator uses the Chef Infra Server API, but only during the first Chef Infra Client run on a node.

Each request to the Chef Infra Server from those executables sign a special group of HTTP headers with the private key. The Chef Infra Server then uses the public key to verify the headers and verify the contents.

.. end_tag

chef-validator
-----------------------------------------------------
.. tag security_chef_validator

Every request made by Chef Infra Client to the Chef Infra Server must be an authenticated request using the Chef Infra Server API and a private key. When Chef Infra Client makes a request to the Chef Infra Server, Chef Infra Client authenticates each request using a private key located in ``/etc/chef/client.pem``.

.. end_tag

.. tag security_chef_validator_context

However, during the first Chef Infra Client run, this private key does not exist. Instead, Chef Infra Client attempts to use the private key assigned to the chef-validator, located in ``/etc/chef/validation.pem``. (If, for any reason, the chef-validator is unable to make an authenticated request to the Chef Infra Server, the initial Chef Infra Client run will fail.)

During the initial Chef Infra Client run, Chef Infra Client registers itself with the Chef Infra Server using the private key assigned to the chef-validator, after which Chef Infra Client will obtain a ``client.pem`` private key for all future authentication requests to the Chef Infra Server.

After the initial Chef Infra Client run has completed successfully, the chef-validator is no longer required and may be deleted from the node. Use the ``delete_validation`` recipe found in the ``chef-client`` cookbook (https://github.com/chef-cookbooks/chef-client) to remove the chef-validator.

.. end_tag

SSL Certificates
=====================================================
An SSL certificate is used between the Chef Infra Client and the Chef Infra Server to ensure that each node has access to the right data.

Signed Headers
-----------------------------------------------------
Signed header authentication is used to validate communications between the Chef Infra Server and any node that is being managed by the Chef Infra Server. An API client manages each authentication request. A public and private key pair is used for the authentication itself. The public key is stored in the database on the Chef Infra Server. The private key is stored locally on each node and is kept separate from node data (typically in the ``/etc/chef/client.pem`` directory). Each request to the Chef Infra Server by a node must include a request signature in the HTTP headers. This signature is computed from a hash of request content and is encrypted using the private key.

During a Chef Infra Client Run
-----------------------------------------------------
.. tag chef_auth_authentication_chef_run

As part of `every Chef Infra Client run </chef_client.html#the-chef-client-run>`_, Chef Infra Client authenticates to the Chef Infra Server using an RSA private key and the Chef Infra Server API.

.. end_tag

SSL Verification
=====================================================
.. warning:: The following information does not apply to hosted Chef Server 12, only to on-premises Chef Server 12.

.. tag server_security_ssl_cert_client

Chef Server 12 enables SSL verification by default for all requests made to the server, such as those made by knife and Chef Infra Client. The certificate that is generated during the installation of the Chef Infra Server is self-signed, which means the certificate is not signed by a trusted certificate authority (CA) that ships with Chef Infra Client. The certificate generated by the Chef Infra Server must be downloaded to any machine from which knife and/or Chef Infra Client will make requests to the Chef Infra Server.

For example, without downloading the SSL certificate, the following knife command:

.. code-block:: bash

   $ knife client list

responds with an error similar to:

.. code-block:: bash

   ERROR: SSL Validation failure connecting to host: chef-server.example.com ...
   ERROR: OpenSSL::SSL::SSLError: SSL_connect returned=1 errno=0 state=SSLv3 ...

This is by design and will occur until a verifiable certificate is added to the machine from which the request is sent.

.. end_tag

``/.chef/trusted_certs``
-----------------------------------------------------
.. tag chef_repo_directory_trusted_certs

The ``/.chef/trusted_certs`` directory stores trusted SSL certificates used to access the Chef Infra Server:

* On each workstation, this directory is the location into which SSL certificates are placed after they are downloaded from the Chef Infra Server using the ``knife ssl fetch`` subcommand
* On every node, this directory is the location into which SSL certificates are placed when a node has been bootstrapped with Chef Infra Client from a workstation

.. end_tag

SSL_CERT_FILE
-----------------------------------------------------
.. tag environment_variables_ssl_cert_file

Use the ``SSL_CERT_FILE`` environment variable to specify the location for the SSL certificate authority (CA) bundle that is used by Chef Infra Client.

A value for ``SSL_CERT_FILE`` is not set by default. Unless updated, the locations in which Chef Infra will look for SSL certificates are:

* Chef Infra Client: ``/opt/chef/embedded/ssl/certs/cacert.pem``
* ChefDK: ``/opt/chefdk/embedded/ssl/certs/cacert.pem``

Keeping the default behavior is recommended. To use a custom CA bundle, update the environment variable to specify the path to the custom CA bundle. If (for some reason) SSL certificate verification stops working, ensure the correct value is specified for ``SSL_CERT_FILE``.

.. end_tag

client.rb Settings
-----------------------------------------------------
.. tag chef_client_ssl_config_settings

Use following client.rb settings to manage SSL certificate preferences:

.. list-table::
   :widths: 200 300
   :header-rows: 1

   * - Setting
     - Description
   * - ``local_key_generation``
     - Whether the Chef Infra Server or Chef Infra Client generates the private/public key pair. When ``true``, Chef Infra Client generates the key pair, and then sends the public key to the Chef Infra Server. Default value: ``true``.
   * - ``ssl_ca_file``
     - The file in which the OpenSSL key is saved. Chef Infra Client generates this setting automatically and most users do not need to modify it.
   * - ``ssl_ca_path``
     - The path to where the OpenSSL key is located. Chef Infra Client generates this setting automatically and most users do not need to modify it.
   * - ``ssl_client_cert``
     - The OpenSSL X.509 certificate used for mutual certificate validation. This setting is only necessary when mutual certificate validation is configured on the Chef Infra Server. Default value: ``nil``.
   * - ``ssl_client_key``
     - The OpenSSL X.509 key used for mutual certificate validation. This setting is only necessary when mutual certificate validation is configured on the Chef Infra Server. Default value: ``nil``.
   * - ``ssl_verify_mode``
     - Set the verify mode for HTTPS requests.

       * Use ``:verify_none`` to do no validation of SSL certificates.
       * Use ``:verify_peer`` to do validation of all SSL certificates, including the Chef Infra Server connections, S3 connections, and any HTTPS **remote_file** resource URLs used in a Chef Infra Client run. This is the recommended setting.

       Depending on how OpenSSL is configured, the ``ssl_ca_path`` may need to be specified. Default value: ``:verify_peer``.
   * - ``verify_api_cert``
     - Verify the SSL certificate on the Chef Infra Server. When ``true``, Chef Infra Client always verifies the SSL certificate. When ``false``, Chef Infra Client uses the value of ``ssl_verify_mode`` to determine if the SSL certificate requires verification. Default value: ``false``.

.. end_tag

Knife Subcommands
-----------------------------------------------------
Chef Infra Client includes two knife commands for managing SSL certificates:

* Use `knife ssl check </knife_ssl_check.html>`__ to troubleshoot SSL certificate issues
* Use `knife ssl fetch </knife_ssl_fetch.html>`__ to pull down a certificate from the Chef Infra Server to the ``/.chef/trusted_certs`` directory on the workstation.

After the workstation has the correct SSL certificate, bootstrap operations from that workstation will use the certificate in the ``/.chef/trusted_certs`` directory during the bootstrap operation.

knife ssl check
+++++++++++++++++++++++++++++++++++++++++++++++++++++
Run the ``knife ssl check`` subcommand to verify the state of the SSL certificate, and then use the reponse to help troubleshoot issues that may be present.

**Verified**

.. tag knife_ssl_check_verify_server_config

If the SSL certificate can be verified, the response to

.. code-block:: bash

   $ knife ssl check

is similar to:

.. code-block:: bash

   Connecting to host chef-server.example.com:443
   Successfully verified certificates from 'chef-server.example.com'

.. end_tag

**Unverified**

.. tag knife_ssl_check_bad_ssl_certificate

If the SSL certificate cannot be verified, the response to

.. code-block:: bash

   $ knife ssl check

is similar to:

.. code-block:: bash

   Connecting to host chef-server.example.com:443
   ERROR: The SSL certificate of chef-server.example.com could not be verified
   Certificate issuer data:
     /C=US/ST=WA/L=S/O=Corp/OU=Ops/CN=chef-server.example.com/emailAddress=you@example.com

   Configuration Info:

   OpenSSL Configuration:
   * Version: OpenSSL 1.0.1j 15 Oct 2014
   * Certificate file: /opt/chefdk/embedded/ssl/cert.pem
   * Certificate directory: /opt/chefdk/embedded/ssl/certs
   Chef SSL Configuration:
   * ssl_ca_path: nil
   * ssl_ca_file: nil
   * trusted_certs_dir: "/Users/grantmc/Downloads/chef-repo/.chef/trusted_certs"

   TO FIX THIS ERROR:

   If the server you are connecting to uses a self-signed certificate,
   you must configure chef to trust that certificate.

   By default, the certificate is stored in the following location on the
   host where your chef-server runs:

     /var/opt/opscode/nginx/ca/SERVER_HOSTNAME.crt

   Copy that file to your trusted_certs_dir (currently:

     /Users/grantmc/Downloads/chef-repo/.chef/trusted_certs)

   using SSH/SCP or some other secure method, then re-run this command to
   confirm that the certificate is now trusted.

.. end_tag

knife ssl fetch
+++++++++++++++++++++++++++++++++++++++++++++++++++++
Run the ``knife ssl fetch`` to download the self-signed certificate from the Chef Infra Server to the ``/.chef/trusted_certs`` directory on a workstation. For example:

.. tag knife_ssl_fetch_verify_certificate

The SSL certificate that is downloaded to the ``/.chef/trusted_certs`` directory should be verified to ensure that it is, in fact, the same certificate as the one located on the Chef Infra Server. This can be done by comparing the SHA-256 checksums.

#. View the checksum on the Chef Infra Server:

   .. code-block:: bash

      $ ssh ubuntu@chef-server.example.com sudo sha256sum /var/opt/opscode/nginx/ca/chef-server.example.com.crt

   The response is similar to:

   .. code-block:: bash

      <ABC123checksum>  /var/opt/opscode/nginx/ca/chef-server.example.com.crt

#. View the checksum on the workstation:

   .. code-block:: bash

      $ gsha256sum .chef/trusted_certs/chef-server.example.com.crt

   The response is similar to:

   .. code-block:: bash

      <ABC123checksum>  .chef/trusted_certs/chef-server.example.com.crt

#. Verify that the checksum values are identical.

.. end_tag

**Verify Checksums**

.. tag knife_ssl_fetch_verify_certificate

The SSL certificate that is downloaded to the ``/.chef/trusted_certs`` directory should be verified to ensure that it is, in fact, the same certificate as the one located on the Chef Infra Server. This can be done by comparing the SHA-256 checksums.

#. View the checksum on the Chef Infra Server:

   .. code-block:: bash

      $ ssh ubuntu@chef-server.example.com sudo sha256sum /var/opt/opscode/nginx/ca/chef-server.example.com.crt

   The response is similar to:

   .. code-block:: bash

      <ABC123checksum>  /var/opt/opscode/nginx/ca/chef-server.example.com.crt

#. View the checksum on the workstation:

   .. code-block:: bash

      $ gsha256sum .chef/trusted_certs/chef-server.example.com.crt

   The response is similar to:

   .. code-block:: bash

      <ABC123checksum>  .chef/trusted_certs/chef-server.example.com.crt

#. Verify that the checksum values are identical.

.. end_tag

Bootstrap Operations
=====================================================

.. tag install_chef_client

The ``knife bootstrap`` command is a common way to install Chef Infra Client on a node. The default for this approach assumes that a node can access the Chef website so that it may download the Chef Infra Client package from that location.

The Chef installer will detect the version of the operating system, and then install the appropriate Chef Infra Client version using a single command to install Chef Infra Client and all of its dependencies, including an embedded version of Ruby, RubyGems, OpenSSL, key-value stores, parsers, libraries, and command line utilities.

The Chef installer puts everything into a unique directory (``/opt/chef/``) so that Chef Infra Client will not interfere with other applications that may be running on the target machine. Once installed, Chef Infra Client requires a few more configuration steps before it can perform its first Chef Infra Client run on a node.

.. end_tag

.. tag chef_client_bootstrap_node

A node is any physical, virtual, or cloud machine that is configured to be maintained by a Chef Infra Client. In order to bootstrap a node, you will first need a working installation of the `Chef software package </packages.html>`__. A bootstrap installs Chef Infra Client on a target system so that it can run as a client and communicate with Chef Infra Server. There are two ways to do this:

* Use the ``knife bootstrap`` subcommand to `bootstrap a node using the Chef installer </install_bootstrap.html>`__
* Use an unattended install to bootstrap a node from itself, without using SSH or WinRM

.. end_tag

.. tag chef_client_bootstrap_stages

The following diagram shows the stages of the bootstrap operation, and then the list below the diagram describes in greater detail each of those stages.

.. image:: ../../images/chef_bootstrap.png

During a ``knife bootstrap`` bootstrap operation, the following happens:

.. list-table::
   :widths: 150 450
   :header-rows: 1

   * - Stages
     - Description
   * - **$ knife bootstrap**
     - On UNIX- and Linux-based machines: The ``knife bootstrap`` subcommand is issued from a workstation. The hostname, IP address, or FQDN of the target node is issued as part of this command. An SSH connection is established with the target node using port 22. A shell script is assembled using the chef-full.erb (the default bootstrap template), and is then executed on the target node.

       On Microsoft Windows machines: The ``knife bootstrap windows winrm`` subcommand is issued from a workstation. (This command is part of the `knife windows plugin </knife_windows.html>`__.) The hostname, IP address, or FQDN of the target node is issued as part of this command. A connection is established with the target node using WinRM over port 5985. (WinRM must be enabled with the corresponding firewall rules in place.)
   * - **Get the install script from Chef**
     - On UNIX- and Linux-based machines: The shell script that is derived from the chef-full.erb bootstrap template will make a request to the Chef website to get the most recent version of a second shell script (``install.sh``).

       On Microsoft Windows machines: The batch file that is derived from the windows-chef-client-msi.erb bootstrap template will make a request to the Chef website to get the .msi installer.
   * - **Get the Chef Infra Client package from Chef**
     - The second shell script (or batch file) then gathers system-specific information and determines the correct package for Chef Infra Client, and then downloads the appropriate package from ``omnitruck-direct.chef.io``.
   * - **Install Chef Infra Client**
     - Chef Infra Client is installed on the target node.
   * - **Start a Chef Infra Client run**
     - On UNIX- and Linux-based machines: The second shell script executes the ``chef-client`` binary with a set of initial settings stored within ``first-boot.json`` on the node. ``first-boot.json`` is generated from the workstation as part of the initial ``knife bootstrap`` subcommand.

       On Microsoft Windows machines: The batch file that is derived from the windows-chef-client-msi.erb bootstrap template executes the ``chef-client`` binary with a set of initial settings stored within ``first-boot.json`` on the node. ``first-boot.json`` is generated from the workstation as part of the initial ``knife bootstrap`` subcommand.
   * - **Complete a Chef Infra Client run**
     - a Chef Infra Client run proceeds, using HTTPS (port 443), and registers the node with the Chef Infra Server.

       The first Chef Infra Client run, by default, contains an empty run-list. A `run-list can be specified </knife_bootstrap.html>`__ as part of the initial bootstrap operation using the ``--run-list`` option as part of the ``knife bootstrap`` subcommand.

.. end_tag
