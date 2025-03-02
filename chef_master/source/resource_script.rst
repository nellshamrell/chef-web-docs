=====================================================
script resource
=====================================================
`[edit on GitHub] <https://github.com/chef/chef-web-docs/blob/master/chef_master/source/resource_script.rst>`__

.. tag resource_script_summary

Use the **script** resource to execute scripts using a specified interpreter, such as Bash, csh, Perl, Python, or Ruby. This resource may also use any of the actions and properties that are available to the **execute** resource. Commands that are executed with this resource are (by their nature) not idempotent, as they are typically unique to the environment in which they are run. Use ``not_if`` and ``only_if`` to guard this resource for idempotence.

.. note:: The **script** resource is different from the **ruby_block** resource because Ruby code that is run with this resource is created as a temporary file and executed like other script resources, rather than run inline.

.. end_tag

This resource is the base resource for several other resources used for scripting on specific platforms. For more information about specific resources for specific platforms, see the following topics:

* `bash </resource_bash.html>`__
* `csh </resource_csh.html>`__
* `ksh </resource_ksh.html>`__
* `perl </resource_perl.html>`__
* `python </resource_python.html>`__
* `ruby </resource_ruby.html>`__

Changed in 12.19 to support windows alternate user identity in execute resources

Syntax
=====================================================
A **script** resource block typically executes scripts using a specified interpreter, such as Bash, csh, Perl, Python, or Ruby:

.. code-block:: ruby

   script 'extract_module' do
     interpreter "bash"
     cwd ::File.dirname(src_filepath)
     code <<-EOH
       mkdir -p #{extract_path}
       tar xzf #{src_filename} -C #{extract_path}
       mv #{extract_path}/*/* #{extract_path}/
       EOH
     not_if { ::File.exist?(extract_path) }
   end

where

* ``interpreter`` specifies the command shell to use
* ``cwd`` specifies the directory from which the command is run
* ``code`` specifies the command to run

It is more common to use the **script**-based resource that is specific to the command shell. Chef has shell-specific resources for Bash, csh, Perl, Python, and Ruby.

The same command as above, but run using the **bash** resource:

.. code-block:: ruby

   bash 'extract_module' do
     cwd ::File.dirname(src_filepath)
     code <<-EOH
       mkdir -p #{extract_path}
       tar xzf #{src_filename} -C #{extract_path}
       mv #{extract_path}/*/* #{extract_path}/
       EOH
     not_if { ::File.exist?(extract_path) }
   end

The full syntax for all of the properties that are available to the **script** resource is:

.. code-block:: ruby

   script 'name' do
     code                       String
     creates                    String
     cwd                        String
     environment                Hash
     flags                      String
     group                      String, Integer
     interpreter                String
     path                       Array
     returns                    Integer, Array
     timeout                    Integer, Float
     user                       String
     password                   String
     domain                     String
     umask                      String, Integer
     action                     Symbol # defaults to :run if not specified
   end

where

* ``script`` is the resource
* ``name`` is the name of the resource block
* ``cwd`` is the location from which the command is run
* ``action`` identifies the steps Chef Infra Client will take to bring the node into the desired state
* ``code``, ``creates``, ``cwd``, ``environment``, ``flags``, ``group``, ``interpreter``, ``path``, ``returns``, ``timeout``, ``user``, ``password``, ``domain`` and ``umask`` are properties of this resource, with the Ruby type shown. See "Properties" section below for more information about all of the properties that may be used with this resource.

Actions
=====================================================

The script resource has the following actions:

``:nothing``
   Prevent a command from running. This action is used to specify that a command is run only when another resource notifies it.

``:run``
   Default. Run a script.

Properties
=====================================================

The script resource has the following properties:

``code``
   **Ruby Type:** String

   A quoted (" ") string of code to be executed.

``creates``
   **Ruby Type:** String

   Prevent a command from creating a file when that file already exists.

``cwd``
   **Ruby Type:** String

   The current working directory from which the command will be run.

``environment``
   **Ruby Type:** Hash

   A Hash of environment variables in the form of ``({"ENV_VARIABLE" => "VALUE"})``. (These variables must exist for a command to be run successfully.)

``flags``
   **Ruby Type:** String

   One or more command line flags that are passed to the interpreter when a command is invoked.

``group``
   **Ruby Type:** String, Integer

   The group name or group ID that must be changed before running a command.

``interpreter``
   **Ruby Type:** String

   The script interpreter to use during code execution.

``path``
   **Ruby Type:** Array

   An array of paths to use when searching for a command. These paths are not added to the command's environment $PATH. The default value uses the system path.

   .. warning:: .. tag resources_common_resource_execute_attribute_path

                The ``path`` property has been deprecated and will throw an exception in Chef Client 12 or later. We recommend you use the ``environment`` property instead.

                .. end_tag

      For example:

      .. code-block:: ruby

         script 'mycommand' do
           environment 'PATH' => "/my/path/to/bin:#{ENV['PATH']}"
         end

``returns``
   **Ruby Type:** Integer, Array | **Default Value:** ``0``

   The return value for a command. This may be an array of accepted values. An exception is raised when the return value(s) do not match.

``timeout``
   **Ruby Type:** Integer, Float | **Default Value:**  ``3600``

   The amount of time (in seconds) a command is to wait before timing out.

``user``
   **Ruby Type:** String

   The user name of the user identity with which to launch the new process. Default value: `nil`. The user name may optionally be specified with a domain, i.e. `domain\user` or `user@my.dns.domain.com` via Universal Principal Name (UPN)format. It can also be specified without a domain simply as user if the domain is instead specified using the `domain` attribute. On Windows only, if this property is specified, the `password` property must be specified.

``password``
   **Ruby Type:** String

   *Windows only*: The password of the user specified by the `user` property.
   Default value: `nil`. This property is mandatory if `user` is specified on Windows and may only be specified if `user` is specified. The `sensitive` property for this resource will automatically be set to true if password is specified.

``domain``
   **Ruby Type:** String

   *Windows only*: The domain of the user user specified by the `user` property.
   Default value: `nil`. If not specified, the user name and password specified by the `user` and `password` properties will be used to resolve that user against the domain in which the system running Chef client is joined, or if that system is not joined to a domain it will resolve the user as a local account on that system. An alternative way to specify the domain is to leave this property unspecified and specify the domain as part of the `user` property.

``umask``
   **Ruby Type:** String, Integer

   The file mode creation mask, or umask.

Common Resource Functionality
=====================================================

Chef resources include common properties, notifications, and resource guards.

Common Properties
-----------------------------------------------------

.. tag resources_common_properties

The following properties are common to every resource:

``ignore_failure``
  **Ruby Type:** true, false | **Default Value:** ``false``

  Continue running a recipe if a resource fails for any reason.

``retries``
  **Ruby Type:** Integer | **Default Value:** ``0``

  The number of attempts to catch exceptions and retry the resource.

``retry_delay``
  **Ruby Type:** Integer | **Default Value:** ``2``

  The retry delay (in seconds).

``sensitive``
  **Ruby Type:** true, false | **Default Value:** ``false``

  Ensure that sensitive resource data is not logged by Chef Infra Client.

.. end_tag

Notifications
-----------------------------------------------------

``notifies``
  **Ruby Type:** Symbol, 'Chef::Resource[String]'

  .. tag resources_common_notification_notifies

  A resource may notify another resource to take action when its state changes. Specify a ``'resource[name]'``, the ``:action`` that resource should take, and then the ``:timer`` for that action. A resource may notify more than one resource; use a ``notifies`` statement for each resource to be notified.

  .. end_tag

.. tag resources_common_notification_timers

A timer specifies the point during a Chef Infra Client run at which a notification is run. The following timers are available:

``:before``
   Specifies that the action on a notified resource should be run before processing the resource block in which the notification is located.

``:delayed``
   Default. Specifies that a notification should be queued up, and then executed at the end of a Chef Infra Client run.

``:immediate``, ``:immediately``
   Specifies that a notification should be run immediately, per resource notified.

.. end_tag

.. tag resources_common_notification_notifies_syntax

The syntax for ``notifies`` is:

.. code-block:: ruby

  notifies :action, 'resource[name]', :timer

.. end_tag

``subscribes``
  **Ruby Type:** Symbol, 'Chef::Resource[String]'

.. tag resources_common_notification_subscribes

A resource may listen to another resource, and then take action if the state of the resource being listened to changes. Specify a ``'resource[name]'``, the ``:action`` to be taken, and then the ``:timer`` for that action.

Note that ``subscribes`` does not apply the specified action to the resource that it listens to - for example:

.. code-block:: ruby

 file '/etc/nginx/ssl/example.crt' do
   mode '0600'
   owner 'root'
 end

 service 'nginx' do
   subscribes :reload, 'file[/etc/nginx/ssl/example.crt]', :immediately
 end

In this case the ``subscribes`` property reloads the ``nginx`` service whenever its certificate file, located under ``/etc/nginx/ssl/example.crt``, is updated. ``subscribes`` does not make any changes to the certificate file itself, it merely listens for a change to the file, and executes the ``:reload`` action for its resource (in this example ``nginx``) when a change is detected.

.. end_tag

.. tag resources_common_notification_timers

A timer specifies the point during a Chef Infra Client run at which a notification is run. The following timers are available:

``:before``
   Specifies that the action on a notified resource should be run before processing the resource block in which the notification is located.

``:delayed``
   Default. Specifies that a notification should be queued up, and then executed at the end of a Chef Infra Client run.

``:immediate``, ``:immediately``
   Specifies that a notification should be run immediately, per resource notified.

.. end_tag

.. tag resources_common_notification_subscribes_syntax

The syntax for ``subscribes`` is:

.. code-block:: ruby

   subscribes :action, 'resource[name]', :timer

.. end_tag

Guards
-----------------------------------------------------

.. tag resources_common_guards

A guard property can be used to evaluate the state of a node during the execution phase of a Chef Infra Client run. Based on the results of this evaluation, a guard property is then used to tell Chef Infra Client if it should continue executing a resource. A guard property accepts either a string value or a Ruby block value:

* A string is executed as a shell command. If the command returns ``0``, the guard is applied. If the command returns any other value, then the guard property is not applied. String guards in a **powershell_script** run Windows PowerShell commands and may return ``true`` in addition to ``0``.
* A block is executed as Ruby code that must return either ``true`` or ``false``. If the block returns ``true``, the guard property is applied. If the block returns ``false``, the guard property is not applied.

A guard property is useful for ensuring that a resource is idempotent by allowing that resource to test for the desired state as it is being executed, and then if the desired state is present, for Chef Infra Client to do nothing.

.. end_tag

**Properties**

.. tag resources_common_guards_properties

The following properties can be used to define a guard that is evaluated during the execution phase of a Chef Infra Client run:

``not_if``
  Prevent a resource from executing when the condition returns ``true``.

``only_if``
  Allow a resource to execute only if the condition returns ``true``.

.. end_tag

Guard Interpreter
-----------------------------------------------------
.. tag resources_common_guard_interpreter

Any resource that passes a string command may also specify the interpreter that will be used to evaluate that string command. This is done by using the ``guard_interpreter`` property to specify a **script**-based resource.

.. end_tag

**Attributes**

.. tag resources_common_guard_interpreter_attributes

The ``guard_interpreter`` property may be set to any of the following values:

``:bash``
   Evaluates a string command using the **bash** resource.

``:batch``
   Evaluates a string command using the **batch** resource. Default value (within a **batch** resource block): ``:batch``.

``:csh``
   Evaluates a string command using the **csh** resource.

``:default``
   Default. Executes the default interpreter as identified by Chef Infra Client.

``:perl``
   Evaluates a string command using the **perl** resource.

``:powershell_script``
   Evaluates a string command using the **powershell_script** resource. Default value (within a **batch** resource block): ``:powershell_script``.

``:python``
   Evaluates a string command using the **python** resource.

``:ruby``
   Evaluates a string command using the **ruby** resource.

.. end_tag

**Inheritance**

.. tag resources_common_guard_interpreter_attributes_inherit

The ``guard_interpreter`` property is set to ``:default`` by default for the **bash**, **csh**, **perl**, **python**, and **ruby** resources. When the ``guard_interpreter`` property is set to ``:default``, ``not_if`` or ``only_if`` guard statements **do not inherit** properties that are defined by the **script**-based resource.

.. warning:: The **batch** and **powershell_script** resources inherit properties by default. The ``guard_interpreter`` property is set to ``:batch`` or ``:powershell_script`` automatically when using a ``not_if`` or ``only_if`` guard statement within a **batch** or **powershell_script** resource, respectively.

For example, the ``not_if`` guard statement in the following resource example **does not inherit** the ``environment`` property:

.. code-block:: ruby

   bash 'javatooling' do
     environment 'JAVA_HOME' => '/usr/lib/java/jdk1.7/home'
     code 'java-based-daemon-ctl.sh -start'
     not_if 'java-based-daemon-ctl.sh -test-started'
   end

and requires adding the ``environment`` property to the ``not_if`` guard statement so that it may use the ``JAVA_HOME`` path as part of its evaluation:

.. code-block:: ruby

   bash 'javatooling' do
     environment 'JAVA_HOME' => '/usr/lib/java/jdk1.7/home'
     code 'java-based-daemon-ctl.sh -start'
     not_if 'java-based-daemon-ctl.sh -test-started', :environment => 'JAVA_HOME' => '/usr/lib/java/jdk1.7/home'
   end

To inherit properties, add the ``guard_interpreter`` property to the resource block and set it to the appropriate value:

* ``:bash`` for **bash**
* ``:csh`` for **csh**
* ``:perl`` for **perl**
* ``:python`` for **python**
* ``:ruby`` for **ruby**

For example, using the same example as from above, but this time adding the ``guard_interpreter`` property and setting it to ``:bash``:

.. code-block:: ruby

   bash 'javatooling' do
     guard_interpreter :bash
     environment 'JAVA_HOME' => '/usr/lib/java/jdk1.7/home'
     code 'java-based-daemon-ctl.sh -start'
     not_if 'java-based-daemon-ctl.sh -test-started'
   end

The ``not_if`` statement now inherits the ``environment`` property and will use the ``JAVA_HOME`` path as part of its evaluation.

.. end_tag

**Example**

.. tag resources_common_guard_interpreter_example_default

For example, the following code block will ensure the command is evaluated using the default interpreter as identified by Chef Infra Client:

.. code-block:: ruby

   resource 'name' do
     guard_interpreter :default
     # code
   end

.. end_tag

Examples
=====================================================
The following examples demonstrate various approaches for using resources in recipes:

**Use a named provider to run a script**

.. tag resource_script_bash_provider_and_interpreter

.. To use the |resource bash| resource to run a script:

.. code-block:: ruby

   bash 'install_something' do
     user 'root'
     cwd '/tmp'
     code <<-EOH
     wget http://www.example.com/tarball.tar.gz
     tar -zxf tarball.tar.gz
     cd tarball
     ./configure
     make
     make install
     EOH
   end

.. end_tag

**Run a script**

.. tag resource_script_bash_script

.. To run a Bash script:

.. code-block:: ruby

   script 'install_something' do
     interpreter 'bash'
     user 'root'
     cwd '/tmp'
     code <<-EOH
     wget http://www.example.com/tarball.tar.gz
     tar -zxf tarball.tar.gz
     cd tarball
     ./configure
     make
     make install
     EOH
   end

or something like:

.. code-block:: ruby

   bash 'openvpn-server-key' do
     environment('KEY_CN' => 'server')
     code <<-EOF
       openssl req -batch -days #{node['openvpn']['key']['expire']} \
         -nodes -new -newkey rsa:#{key_size} -keyout #{key_dir}/server.key \
         -out #{key_dir}/server.csr -extensions server \
         -config #{key_dir}/openssl.cnf
     EOF
     not_if { File.exist?('#{key_dir}/server.crt') }
   end

where ``code`` contains the OpenSSL command to be run. The ``not_if`` property tells Chef Infra Client not to run the command if the file already exists.

.. end_tag

**Install a file from a remote location using bash**

.. tag resource_remote_file_install_with_bash

The following is an example of how to install the ``foo123`` module for Nginx. This module adds shell-style functionality to an Nginx configuration file and does the following:

* Declares three variables
* Gets the Nginx file from a remote location
* Installs the file using Bash to the path specified by the ``src_filepath`` variable

.. code-block:: ruby

   # the following code sample is similar to the ``upload_progress_module``
   # recipe in the ``nginx`` cookbook:
   # https://github.com/chef-cookbooks/nginx

   src_filename = "foo123-nginx-module-v#{
     node['nginx']['foo123']['version']
   }.tar.gz"
   src_filepath = "#{Chef::Config['file_cache_path']}/#{src_filename}"
   extract_path = "#{
     Chef::Config['file_cache_path']
     }/nginx_foo123_module/#{
     node['nginx']['foo123']['checksum']
   }"

   remote_file 'src_filepath' do
     source node['nginx']['foo123']['url']
     checksum node['nginx']['foo123']['checksum']
     owner 'root'
     group 'root'
     mode '0755'
   end

   bash 'extract_module' do
     cwd ::File.dirname(src_filepath)
     code <<-EOH
       mkdir -p #{extract_path}
       tar xzf #{src_filename} -C #{extract_path}
       mv #{extract_path}/*/* #{extract_path}/
       EOH
     not_if { ::File.exist?(extract_path) }
   end

.. end_tag

**Install an application from git using bash**

.. tag resource_scm_use_bash_and_ruby_build

The following example shows how Bash can be used to install a plug-in for rbenv named ``ruby-build``, which is located in git version source control. First, the application is synchronized, and then Bash changes its working directory to the location in which ``ruby-build`` is located, and then runs a command.

.. code-block:: ruby

   git "#{Chef::Config[:file_cache_path]}/ruby-build" do
     repository 'git://github.com/sstephenson/ruby-build.git'
     revision 'master'
     action :sync
   end

   bash 'install_ruby_build' do
     cwd '#{Chef::Config[:file_cache_path]}/ruby-build'
     user 'rbenv'
     group 'rbenv'
     code <<-EOH
       ./install.sh
       EOH
     environment 'PREFIX' => '/usr/local'
  end

To read more about ``ruby-build``, see here: https://github.com/sstephenson/ruby-build.

.. end_tag

**Store certain settings**

.. tag resource_remote_file_store_certain_settings

The following recipe shows how an attributes file can be used to store certain settings. An attributes file is located in the ``attributes/`` directory in the same cookbook as the recipe which calls the attributes file. In this example, the attributes file specifies certain settings for Python that are then used across all nodes against which this recipe will run.

Python packages have versions, installation directories, URLs, and checksum files. An attributes file that exists to support this type of recipe would include settings like the following:

.. code-block:: ruby

   default['python']['version'] = '2.7.1'

   if python['install_method'] == 'package'
     default['python']['prefix_dir'] = '/usr'
   else
     default['python']['prefix_dir'] = '/usr/local'
   end

   default['python']['url'] = 'http://www.python.org/ftp/python'
   default['python']['checksum'] = '80e387...85fd61'

and then the methods in the recipe may refer to these values. A recipe that is used to install Python will need to do the following:

* Identify each package to be installed (implied in this example, not shown)
* Define variables for the package ``version`` and the ``install_path``
* Get the package from a remote location, but only if the package does not already exist on the target system
* Use the **bash** resource to install the package on the node, but only when the package is not already installed

.. code-block:: ruby

   #  the following code sample comes from the ``oc-nginx`` cookbook on |github|: https://github.com/cookbooks/oc-nginx

   version = node['python']['version']
   install_path = "#{node['python']['prefix_dir']}/lib/python#{version.split(/(^\d+\.\d+)/)[1]}"

   remote_file "#{Chef::Config[:file_cache_path]}/Python-#{version}.tar.bz2" do
     source "#{node['python']['url']}/#{version}/Python-#{version}.tar.bz2"
     checksum node['python']['checksum']
     mode '0755'
     not_if { ::File.exist?(install_path) }
   end

   bash 'build-and-install-python' do
     cwd Chef::Config[:file_cache_path]
     code <<-EOF
       tar -jxvf Python-#{version}.tar.bz2
       (cd Python-#{version} && ./configure #{configure_options})
       (cd Python-#{version} && make && make install)
     EOF
     not_if { ::File.exist?(install_path) }
   end

.. end_tag

**Run a command as an alternate user**

*Note*: When Chef is running as a service, this feature requires that the user that Chef runs as has 'SeAssignPrimaryTokenPrivilege' (aka 'SE_ASSIGNPRIMARYTOKEN_NAME') user right. By default only LocalSystem and NetworkService have this right when running as a service. This is necessary even if the user is an Administrator.

This right can be added and checked in a recipe using this example:

.. code-block:: ruby

    # Add 'SeAssignPrimaryTokenPrivilege' for the user
    Chef::ReservedNames::Win32::Security.add_account_right('<user>', 'SeAssignPrimaryTokenPrivilege')

    # Check if the user has 'SeAssignPrimaryTokenPrivilege' rights
    Chef::ReservedNames::Win32::Security.get_account_right('<user>').include?('SeAssignPrimaryTokenPrivilege')

The following example shows how to run ``mkdir test_dir`` from a Chef Infra Client run as an alternate user.

.. code-block:: ruby

   # Passing only username and password
   script 'mkdir test_dir' do
    interpreter "bash"
    code  "mkdir test_dir"
    cwd Chef::Config[:file_cache_path]
    user "username"
    password "password"
   end

   # Passing username and domain
   script 'mkdir test_dir' do
    interpreter "bash"
    code  "mkdir test_dir"
    cwd Chef::Config[:file_cache_path]
    domain "domain-name"
    user "username"
    password "password"
   end

   # Passing username = 'domain-name\\username'. No domain is passed
   script 'mkdir test_dir' do
    interpreter "bash"
    code  "mkdir test_dir"
    cwd Chef::Config[:file_cache_path]
    user "domain-name\\username"
    password "password"
   end

   # Passing username = 'username@domain-name'. No domain is passed
   script 'mkdir test_dir' do
    interpreter "bash"
    code  "mkdir test_dir"
    cwd Chef::Config[:file_cache_path]
    user "username@domain-name"
    password "password"
   end


