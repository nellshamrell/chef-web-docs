========================================================
Data Collection with a Chef High Availability Cluster
========================================================
`[edit on GitHub] <https://github.com/chef/chef-web-docs/blob/master/chef_master/source/data_collection_ha.rst>`__

.. meta:: 
    :robots: noindex 

.. important:: Previous step: `Setup Data Collection </data_collection.html>`__

To configure front-end servers in your HA cluster to send their object data, first configure a Chef Infra Server for `data collection </data_collection.html>`__ and ensure that the ``fqdn`` field in all of your front-end Chef Infra Server ``/etc/opscode/chef-server.rb`` files are the same.

The following example sets the ``fqdn`` field to ``"my-chef-server.mycompany.com"`` in two front-end servers.

**chef-server.rb.FE1**

.. code-block:: ruby

  # This file generated by chef-backend-ctl gen-server-config
  # Modify with extreme caution.
  fqdn "my-chef-server.mycompany.com"
  use_chef_backend true
  data_collector['root_url'] = 'https://my-automate-server.mycompany.com/data-collector/v0/'
  data_collector['token'] = 'TOKEN'

**chef-server.rb.FE2**

.. code-block:: ruby

  # This file generated by chef-backend-ctl gen-server-config
  # Modify with extreme caution.
  fqdn "my-chef-server.mycompany.com"
  use_chef_backend true
  data_collector['root_url'] = 'https://my-automate-server.mycompany.com/data-collector/v0/'
  data_collector['token'] = 'TOKEN'

.. warning:: Failure to set the ``fqdn`` field to the same value will result in Chef Automate treating data from each of these front-end servers as separate Chef servers.

Next Steps
============================
   * `Perform a Compliance Scan </perform_compliance_scan.html>`__
   * `Data Collection  </data_collection.html>`__
   * `Data Collection without Chef Infra Server </data_collection_without_server.html>`__
