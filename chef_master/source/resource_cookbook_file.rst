=====================================================
cookbook_file resource
=====================================================
`[edit on GitHub] <https://github.com/chef/chef-web-docs/blob/master/chef_master/source/resource_cookbook_file.rst>`__

.. tag resource_cookbook_file_summary

Use the **cookbook_file** resource to transfer files from a sub-directory of ``COOKBOOK_NAME/files/`` to a specified path located on a host that is running Chef Infra Client. The file is selected according to file specificity, which allows different source files to be used based on the hostname, host platform (operating system, distro, or as appropriate), or platform version. Files that are located in the ``COOKBOOK_NAME/files/default`` sub-directory may be used on any platform.

.. end_tag

During a Chef Infra Client run, the checksum for each local file is calculated and then compared against the checksum for the same file as it currently exists in the cookbook on the Chef server. A file is not transferred when the checksums match. Only files that require an update are transferred from the Chef server to a node.

Syntax
=====================================================
A **cookbook_file** resource block manages files by using files that exist within a cookbook's ``/files`` directory. For example, to write the home page for an Apache website:

.. code-block:: ruby

   cookbook_file '/var/www/customers/public_html/index.php' do
     source 'index.php'
     owner 'web_admin'
     group 'web_admin'
     mode '0755'
     action :create
   end

where

* ``'/var/www/customers/public_html/index.php'`` is path to the file to be created
* ``'index.php'`` is a file in the ``/files`` directory in a cookbook that is used to create that file (the contents of the file in the cookbook will become the contents of the file on the node)
* ``owner``, ``group``, and ``mode`` define the permissions

The full syntax for all of the properties that are available to the **cookbook_file** resource is:

.. code-block:: ruby

  cookbook_file 'name' do
    atomic_update              true, false
    backup                     Integer, false # default value: 5
    cookbook                   String
    force_unlink               true, false # default value: false
    group                      String, Integer
    inherits                   true, false
    manage_symlink_source      true, false
    mode                       String, Integer
    owner                      String, Integer
    path                       String # default value: 'name' unless specified
    rights                     Hash
    source                     String, Array
    verify                     String, Block
    action                     Symbol # defaults to :create if not specified
  end

where:

* ``cookbook_file`` is the resource.
* ``name`` is the name given to the resource block.
* ``action`` identifies which steps Chef Infra Client will take to bring the node into the desired state.
* ``atomic_update``, ``backup``, ``cookbook``, ``force_unlink``, ``group``, ``inherits``, ``manage_symlink_source``, ``mode``, ``owner``, ``path``, ``rights``, ``source``, and ``verify`` are properties of this resource, with the Ruby type shown. See "Properties" section below for more information about all of the properties that may be used with this resource.

Actions
=====================================================

The cookbook_file resource has the following actions:

``:create``
   Default. Create a file. If a file already exists (but does not match), update that file to match.

``:create_if_missing``
   Create a file only if the file does not exist. When the file exists, nothing happens.

``:delete``
   Delete a file.

``:nothing``
   .. tag resources_common_actions_nothing

   This resource block does not act unless notified by another resource to take action. Once notified, this resource block either runs immediately or is queued up to run at the end of a Chef Infra Client run.

   .. end_tag

``:touch``
   Touch a file. This updates the access (atime) and file modification (mtime) times for a file. (This action may be used with this resource, but is typically only used with the **file** resource.)

Properties
=====================================================

The cookbook_file resource has the following properties:

``atomic_update``
   **Ruby Type:** true, false

   Perform atomic file updates on a per-resource basis. Set to ``true`` for atomic file updates. Set to ``false`` for non-atomic file updates. This setting overrides ``file_atomic_update``, which is a global setting found in the client.rb file.

``backup``
   **Ruby Type:** Integer, false | **Default Value:** ``5``

   The number of backups to be kept in ``/var/chef/backup`` (for UNIX- and Linux-based platforms) or ``C:/chef/backup`` (for the Microsoft Windows platform). Set to ``false`` to prevent backups from being kept.

``cookbook``
   **Ruby Type:** String

   The cookbook in which a file is located (if it is not located in the current cookbook). The default value is the current cookbook.

``force_unlink``
   **Ruby Type:** true, false | **Default Value:** ``false``

   How Chef Infra Client handles certain situations when the target file turns out not to be a file. For example, when a target file is actually a symlink. Set to ``true`` for Chef Infra Client delete the non-file target and replace it with the specified file. Set to ``false`` for Chef Infra Client to raise an error.

``group``
   **Ruby Type:** Integer, String

   A string or ID that identifies the group owner by group name, including fully qualified group names such as ``domain\group`` or ``group@domain``. If this value is not specified, existing groups remain unchanged and new group assignments use the default ``POSIX`` group (if available).

``inherits``
   **Ruby Type:** true, false | **Default Value:** ``true``

   Microsoft Windows only. Whether a file inherits rights from its parent directory.

``manage_symlink_source``
   **Ruby Type:** true, false | **Default Value:** ``true`` (with warning)

   Change the behavior of the file resource if it is pointed at a symlink. When this value is set to ``true``, Chef Infra Client will manage the symlink's permissions or will replace the symlink with a normal file if the resource has content. When this value is set to ``false``, Chef will follow the symlink and will manage the permissions and content of the symlink's target file.

   The default behavior is ``true`` but emits a warning that the default value will be changed to ``false`` in a future version; setting this explicitly to ``true`` or ``false`` suppresses this warning.

``mode``
   **Ruby Type:** Integer, String

   If ``mode`` is not specified and if the file already exists, the existing mode on the file is used. If ``mode`` is not specified, the file does not exist, and the ``:create`` action is specified, Chef Infra Client assumes a mask value of ``'0777'`` and then applies the umask for the system on which the file is to be created to the ``mask`` value. For example, if the umask on a system is ``'022'``, Chef Infra Client uses the default value of ``'0755'``.

   The behavior is different depending on the platform.

   UNIX- and Linux-based systems: A quoted 3-5 character string that defines the octal mode that is passed to chmod. For example: ``'755'``, ``'0755'``, or ``00755``. If the value is specified as a quoted string, it works exactly as if the ``chmod`` command was passed. If the value is specified as an integer, prepend a zero (``0``) to the value to ensure that it is interpreted as an octal number. For example, to assign read, write, and execute rights for all users, use ``'0777'`` or ``'777'``; for the same rights, plus the sticky bit, use ``01777`` or ``'1777'``.

   Microsoft Windows: A quoted 3-5 character string that defines the octal mode that is translated into rights for Microsoft Windows security. For example: ``'755'``, ``'0755'``, or ``00755``. Values up to ``'0777'`` are allowed (no sticky bits) and mean the same in Microsoft Windows as they do in UNIX, where ``4`` equals ``GENERIC_READ``, ``2`` equals ``GENERIC_WRITE``, and ``1`` equals ``GENERIC_EXECUTE``. This property cannot be used to set ``:full_control``. This property has no effect if not specified, but when it and ``rights`` are both specified, the effects are cumulative.

``owner``
   **Ruby Type:** Integer, String

   A string or ID that identifies the group owner by user name, including fully qualified user names such as ``domain\user`` or ``user@domain``. If this value is not specified, existing owners remain unchanged and new owner assignments use the current user (when necessary).

``path``
   **Ruby Type:** String

   The path to the destination at which a file is to be created. Default value: the ``name`` of the resource block For example: ``file.txt``.

   Microsoft Windows: A path that begins with a forward slash (``/``) will point to the root of the current working directory of Chef Infra Client process. This path can vary from system to system. Therefore, using a path that begins with a forward slash (``/``) is not recommended.

``rights``
   **Ruby Type:** Integer, String

   Microsoft Windows only. The permissions for users and groups in a Microsoft Windows environment. For example: ``rights <permissions>, <principal>, <options>`` where ``<permissions>`` specifies the rights granted to the principal, ``<principal>`` is the group or user name, and ``<options>`` is a Hash with one (or more) advanced rights options.

``source``
   **Ruby Type:** String, Array | **Default Value:** ``The resource block's name``

   The name of the file in ``COOKBOOK_NAME/files/default`` or the path to a file located in ``COOKBOOK_NAME/files``. The path must include the file name and its extension. This can be used to distribute specific files depending upon the platform used - see `File Specificity <#file-specificity>`_ for more information.

``verify``
   **Ruby Type:** String, Block

   A block or a string that returns ``true`` or ``false``. A string, when ``true`` is executed as a system command.

   A block is arbitrary Ruby defined within the resource block by using the ``verify`` property. When a block is ``true``, Chef Infra Client will continue to update the file as appropriate.

   For example, this should return ``true``:

   .. code-block:: ruby

      cookbook_file '/tmp/baz' do
        verify { 1 == 1 }
      end

   This should return ``true``:

   .. code-block:: ruby

      cookbook_file '/etc/nginx.conf' do
        verify 'nginx -t -c %{path}'
      end

   .. warning:: For releases of Chef Infra Client prior to 12.5 (chef-client 12.4 and earlier) the correct syntax is:

      .. code-block:: ruby

         cookbook_file '/etc/nginx.conf' do
           verify 'nginx -t -c %{file}'
         end

      See GitHub issues https://github.com/chef/chef/issues/3232 and https://github.com/chef/chef/pull/3693 for more information about these differences.

   This should return ``true``:

   .. code-block:: ruby

      cookbook_file '/tmp/bar' do
        verify { 1 == 1}
      end

   And this should return ``true``:

   .. code-block:: ruby

      cookbook_file '/tmp/foo' do
        verify do |path|
          true
        end
      end

   Whereas, this should return ``false``:

   .. code-block:: ruby

      cookbook_file '/tmp/turtle' do
        verify '/usr/bin/false'
      end

   If a string or a block return ``false``, the Chef Infra Client run will stop and an error is returned.

.. note:: Use the ``owner`` and ``right`` properties and avoid the ``group`` and ``mode`` properties whenever possible. The ``group`` and ``mode`` properties are not true Microsoft Windows concepts and are provided more for backward compatibility than for best practice.

Atomic File Updates
-----------------------------------------------------
.. tag resources_common_atomic_update

Atomic updates are used with **file**-based resources to help ensure that file updates can be made when updating a binary or if disk space runs out.

Atomic updates are enabled by default. They can be managed globally using the ``file_atomic_update`` setting in the client.rb file. They can be managed on a per-resource basis using the ``atomic_update`` property that is available with the **cookbook_file**, **file**, **remote_file**, and **template** resources.

.. note:: On certain platforms, and after a file has been moved into place, Chef Infra Client may modify file permissions to support features specific to those platforms. On platforms with SELinux enabled, Chef Infra Client will fix up the security contexts after a file has been moved into the correct location by running the ``restorecon`` command. On the Microsoft Windows platform, Chef Infra Client will create files so that ACL inheritance works as expected.

.. end_tag

Windows File Security
-----------------------------------------------------
.. tag resources_common_windows_security

To support Microsoft Windows security, the **template**, **file**, **remote_file**, **cookbook_file**, **directory**, and **remote_directory** resources support the use of inheritance and access control lists (ACLs) within recipes.

.. end_tag

**Access Control Lists (ACLs)**

.. tag resources_common_windows_security_acl

The ``rights`` property can be used in a recipe to manage access control lists (ACLs), which allow permissions to be given to multiple users and groups. Use the ``rights`` property can be used as many times as necessary; Chef Infra Client will apply them to the file or directory as required. The syntax for the ``rights`` property is as follows:

.. code-block:: ruby

   rights permission, principal, option_type => value

where

``permission``
   Use to specify which rights are granted to the ``principal``. The possible values are: ``:read``, ``:write``, ``read_execute``, ``:modify``, and ``:full_control``.

   These permissions are cumulative. If ``:write`` is specified, then it includes ``:read``. If ``:full_control`` is specified, then it includes both ``:write`` and ``:read``.

   (For those who know the Microsoft Windows API: ``:read`` corresponds to ``GENERIC_READ``; ``:write`` corresponds to ``GENERIC_WRITE``; ``:read_execute`` corresponds to ``GENERIC_READ`` and ``GENERIC_EXECUTE``; ``:modify`` corresponds to ``GENERIC_WRITE``, ``GENERIC_READ``, ``GENERIC_EXECUTE``, and ``DELETE``; ``:full_control`` corresponds to ``GENERIC_ALL``, which allows a user to change the owner and other metadata about a file.)

``principal``
   Use to specify a group or user name. This is identical to what is entered in the login box for Microsoft Windows, such as ``user_name``, ``domain\user_name``, or ``user_name@fully_qualified_domain_name``. Chef Infra Client does not need to know if a principal is a user or a group.

``option_type``
   A hash that contains advanced rights options. For example, the rights to a directory that only applies to the first level of children might look something like: ``rights :write, 'domain\group_name', :one_level_deep => true``. Possible option types:

   .. list-table::
      :widths: 60 420
      :header-rows: 1

      * - Option Type
        - Description
      * - ``:applies_to_children``
        - Specify how permissions are applied to children. Possible values: ``true`` to inherit both child directories and files;  ``false`` to not inherit any child directories or files; ``:containers_only`` to inherit only child directories (and not files); ``:objects_only`` to recursively inherit files (and not child directories).
      * - ``:applies_to_self``
        - Indicates whether a permission is applied to the parent directory. Possible values: ``true`` to apply to the parent directory or file and its children; ``false`` to not apply only to child directories and files.
      * - ``:one_level_deep``
        - Indicates the depth to which permissions will be applied. Possible values: ``true`` to apply only to the first level of children; ``false`` to apply to all children.

For example:

.. code-block:: ruby

   resource 'x.txt' do
     rights :read, 'Everyone'
     rights :write, 'domain\group'
     rights :full_control, 'group_name_or_user_name'
     rights :full_control, 'user_name', :applies_to_children => true
   end

or:

.. code-block:: ruby

    rights :read, ['Administrators','Everyone']
    rights :full_control, 'Users', :applies_to_children => true
    rights :write, 'Sally', :applies_to_children => :containers_only, :applies_to_self => false, :one_level_deep => true

Some other important things to know when using the ``rights`` attribute:

* Only inherited rights remain. All existing explicit rights on the object are removed and replaced.
* If rights are not specified, nothing will be changed. Chef Infra Client does not clear out the rights on a file or directory if rights are not specified.
* Changing inherited rights can be expensive. Microsoft Windows will propagate rights to all children recursively due to inheritance. This is a normal aspect of Microsoft Windows, so consider the frequency with which this type of action is necessary and take steps to control this type of action if performance is the primary consideration.

Use the ``deny_rights`` property to deny specific rights to specific users. The ordering is independent of using the ``rights`` property. For example, it doesn't matter if rights are granted to everyone is placed before or after ``deny_rights :read, ['Julian', 'Lewis']``, both Julian and Lewis will be unable to read the document. For example:

.. code-block:: ruby

   resource 'x.txt' do
     rights :read, 'Everyone'
     rights :write, 'domain\group'
     rights :full_control, 'group_name_or_user_name'
     rights :full_control, 'user_name', :applies_to_children => true
     deny_rights :read, ['Julian', 'Lewis']
   end

or:

.. code-block:: ruby

   deny_rights :full_control, ['Sally']

.. end_tag

**Inheritance**

.. tag resources_common_windows_security_inherits

By default, a file or directory inherits rights from its parent directory. Most of the time this is the preferred behavior, but sometimes it may be necessary to take steps to more specifically control rights. The ``inherits`` property can be used to specifically tell Chef Infra Client to apply (or not apply) inherited rights from its parent directory.

For example, the following example specifies the rights for a directory:

.. code-block:: ruby

   directory 'C:\mordor' do
     rights :read, 'MORDOR\Minions'
     rights :full_control, 'MORDOR\Sauron'
   end

and then the following example specifies how to use inheritance to deny access to the child directory:

.. code-block:: ruby

   directory 'C:\mordor\mount_doom' do
     rights :full_control, 'MORDOR\Sauron'
     inherits false # Sauron is the only person who should have any sort of access
   end

If the ``deny_rights`` permission were to be used instead, something could slip through unless all users and groups were denied.

Another example also shows how to specify rights for a directory:

.. code-block:: ruby

   directory 'C:\mordor' do
     rights :read, 'MORDOR\Minions'
     rights :full_control, 'MORDOR\Sauron'
     rights :write, 'SHIRE\Frodo' # Who put that there I didn't put that there
   end

but then not use the ``inherits`` property to deny those rights on a child directory:

.. code-block:: ruby

   directory 'C:\mordor\mount_doom' do
     deny_rights :read, 'MORDOR\Minions' # Oops, not specific enough
   end

Because the ``inherits`` property is not specified, Chef Infra Client will default it to ``true``, which will ensure that security settings for existing files remain unchanged.

.. end_tag


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

File Specificity
=====================================================
A cookbook is frequently designed to work across many platforms and is often required to distribute a specific file to a specific platform. A cookbook can be designed to support the distribution of files across platforms, while ensuring that the correct file ends up on each system.

The pattern for file specificity depends on two things: the lookup path and the source attribute. The first pattern that matches is used:

#. /host-$fqdn/$source
#. /$platform-$platform_version/$source
#. /$platform/$source
#. /default/$source
#. /$source

Use an array with the ``source`` attribute to define an explicit lookup path. For example:

.. code-block:: ruby

   file '/conf.py' do
     source ['#{node.chef_environment}.py', 'conf.py']
   end

The following example emulates the entire file specificity pattern by defining it as an explicit path:

.. code-block:: ruby

   file '/conf.py' do
     source %W{
       host-#{node['fqdn']}/conf.py
       #{node['platform']}-#{node['platform_version']}/conf.py
       #{node['platform']}/conf.py
       default/conf.py
     }
   end

A cookbook may have a ``/files`` directory structure like this::

   files/
      host-foo.example.com
      ubuntu-16.04
      ubuntu-16
      ubuntu
      redhat-5.11
      redhat-6.9
      ...
      default

and a resource that looks something like the following:

.. code-block:: ruby

   cookbook_file '/usr/local/bin/apache2_module_conf_generate.pl' do
     source 'apache2_module_conf_generate.pl'
     mode '0755'
     owner 'root'
     group 'root'
   end

This resource is matched in the same order as the ``/files`` directory structure. For a node that is running Ubuntu 16.04, the second item would be the matching item and the location to which the file identified in the **cookbook_file** resource would be distributed:

.. code-block:: ruby

   host-foo.example.com/apache2_module_conf_generate.pl
   ubuntu-16.04/apache2_module_conf_generate.pl
   ubuntu-16/apache2_module_conf_generate.pl
   ubuntu/apache2_module_conf_generate.pl
   default/apache2_module_conf_generate.pl

If the ``apache2_module_conf_generate.pl`` file was located in the cookbook directory under ``files/host-foo.example.com/``, the specified file(s) would only be copied to the machine with the domain name foo.example.com.

**Host Notation**

The naming of folders within cookbook directories must literally match the host notation used for file specificity matching. For example, if a host is named ``foo.example.com``, the folder must be named ``host-foo.example.com``.

Examples
=====================================================
The following examples demonstrate various approaches for using resources in recipes:

**Transfer a file**

.. tag resource_cookbook_file_transfer_a_file

.. To transfer a file in a cookbook:

.. code-block:: ruby

   cookbook_file 'file.txt' do
     mode '0755'
   end

.. end_tag

**Handle cookbook_file and package resources in the same recipe**

.. tag resource_package_handle_cookbook_file_and_package

.. To handle cookbook_file and package when both called in the same recipe

When a **cookbook_file** resource and a **package** resource are both called from within the same recipe, use the ``flush_cache`` attribute to dump the in-memory Yum cache, and then use the repository immediately to ensure that the correct package is installed:

.. code-block:: ruby

   cookbook_file '/etc/yum.repos.d/custom.repo' do
     source 'custom'
     mode '0755'
   end

   package 'only-in-custom-repo' do
     action :install
     flush_cache [ :before ]
   end

.. end_tag

**Install repositories from a file, trigger a command, and force the internal cache to reload**

.. tag resource_package_install_yum_repo_from_file

The following example shows how to install new Yum repositories from a file, where the installation of the repository triggers a creation of the Yum cache that forces the internal cache for Chef Infra Client to reload:

.. code-block:: ruby

   execute 'create-yum-cache' do
    command 'yum -q makecache'
    action :nothing
   end

   ruby_block 'reload-internal-yum-cache' do
     block do
       Chef::Provider::Package::Yum::YumCache.instance.reload
     end
     action :nothing
   end

   cookbook_file '/etc/yum.repos.d/custom.repo' do
     source 'custom'
     mode '0755'
     notifies :run, 'execute[create-yum-cache]', :immediately
     notifies :create, 'ruby_block[reload-internal-yum-cache]', :immediately
   end

.. end_tag

**Use a case statement**

.. tag resource_cookbook_file_use_case_statement

The following example shows how a case statement can be used to handle a situation where an application needs to be installed on multiple platforms, but where the install directories are different paths, depending on the platform:

.. code-block:: ruby

   cookbook_file 'application.pm' do
     path case node['platform']
       when 'centos','redhat'
         '/usr/lib/version/1.2.3/dir/application.pm'
       when 'arch'
         '/usr/share/version/core_version/dir/application.pm'
       else
         '/etc/version/dir/application.pm'
       end
     source "application-#{node['languages']['perl']['version']}.pm"
     owner 'root'
     group 'root'
     mode '0755'
   end

.. end_tag

**Manage dotfiles**

.. tag resource_directory_manage_dotfiles

The following example shows using the **directory** and **cookbook_file** resources to manage dotfiles. The dotfiles are defined by a JSON data structure similar to:

.. code-block:: javascript

   "files": {
     ".zshrc": {
       "mode": '0755',
       "source": "dot-zshrc"
       },
     ".bashrc": {
       "mode": '0755',
       "source": "dot-bashrc"
        },
     ".bash_profile": {
       "mode": '0755',
       "source": "dot-bash_profile"
       },
     }

and then the following resources manage the dotfiles:

.. code-block:: ruby

   if u.has_key?('files')
     u['files'].each do |filename, file_data|

     directory "#{home_dir}/#{File.dirname(filename)}" do
       recursive true
       mode '0755'
     end if file_data['subdir']

     cookbook_file "#{home_dir}/#{filename}" do
       source "#{u['id']}/#{file_data['source']}"
       owner 'u['id']'
       group 'group_id'
       mode 'file_data['mode']'
       ignore_failure true
       backup 0
     end
   end

.. end_tag
