=====================================================
About Handlers
=====================================================
`[edit on GitHub] <https://github.com/chef/chef-web-docs/blob/master/chef_master/source/handlers.rst>`__

.. tag handler

Use a handler to identify situations that arise during a Chef Infra Client run, and then tell Chef Infra Client how to handle these situations when they occur.

.. end_tag

.. tag handler_types

There are three types of handlers:

.. list-table::
   :widths: 60 420
   :header-rows: 1

   * - Handler
     - Description
   * - exception
     - An exception handler is used to identify situations that have caused a Chef Infra Client run to fail. An exception handler can be loaded at the start of a Chef Infra Client run by adding a recipe that contains the **chef_handler** resource to a node's run-list. An exception handler runs when the ``failed?`` property for the ``run_status`` object returns ``true``.
   * - report
     - A report handler is used when a Chef Infra Client run succeeds and reports back on certain details about that Chef Infra Client run. A report handler can be loaded at the start of a Chef Infra Client run by adding a recipe that contains the **chef_handler** resource to a node's run-list. A report handler runs when the ``success?`` property for the ``run_status`` object returns ``true``.
   * - start
     - A start handler is used to run events at the beginning of a Chef Infra Client run. A start handler can be loaded at the start of a Chef Infra Client run by adding the start handler to the ``start_handlers`` setting in the client.rb file or by installing the gem that contains the start handler by using the **chef_gem** resource in a recipe in the **chef-client** cookbook. (A start handler may not be loaded using the ``chef_handler`` resource.)

.. end_tag

Exception/Report Handlers
=====================================================
.. tag handler_type_exception_report

Exception and report handlers are used to trigger certain behaviors in response to specific situations, typically identified during a Chef Infra Client run.

* An exception handler is used to trigger behaviors when a defined aspect of a Chef Infra Client run fails.
* A report handler is used to trigger behaviors when a defined aspect of a Chef Infra Client run is successful.

Both types of handlers can be used to gather data about a Chef Infra Client run and can provide rich levels of data about all types of usage, which can be used later for trending and analysis across the entire organization.

Exception and report handlers are made available to a Chef Infra Client run in one of the following ways:

* By adding the **chef_handler** resource to a recipe, and then adding that recipe to the run-list for a node. (The **chef_handler** resource is available from the **chef_handler** cookbook.)
* By adding the handler to one of the following settings in the node's client.rb file: ``exception_handlers`` and/or ``report_handlers``

.. end_tag

Run from Recipes
-----------------------------------------------------
.. tag handler_type_exception_report_run_from_recipe

The **chef_handler** resource allows exception and report handlers to be enabled from within recipes, which can then added to the run-list for any node on which the exception or report handler should run. The **chef_handler** resource is available from the **chef_handler** cookbook.

To use the **chef_handler** resource in a recipe, add code similar to the following:

.. code-block:: ruby

   chef_handler 'name_of_handler' do
     source '/path/to/handler/handler_name'
     action :enable
   end

For example, a handler for Growl needs to be enabled at the beginning of a Chef Infra Client run:

.. code-block:: ruby

   chef_gem 'chef-handler-growl'

and then is activated in a recipe by using the **chef_handler** resource:

.. code-block:: ruby

   chef_handler 'Chef::Handler::Growl' do
     source 'chef/handler/growl'
     action :enable
   end

.. end_tag

Run from client.rb
-----------------------------------------------------
A simple exception or report handler may be installed and configured at run-time. This requires editing of a node's client.rb file to add the appropriate setting and information about that handler to the client.rb or solo.rb files. Depending on the handler type, one (or more) of the following settings must be added:

.. list-table::
   :widths: 200 300
   :header-rows: 1

   * - Setting
     - Description
   * - ``exception_handlers``
     - A list of exception handlers that are available to Chef Infra Client during a Chef Infra Client run.
   * - ``report_handlers``
     - A list of report handlers that are available to Chef Infra Client during a Chef Infra Client run.

When this approach is used, the client.rb file must also tell Chef Infra Client how to install and run the handler. There is no default install location for handlers. The simplest way to distribute and install them is via RubyGems, though other methods such as GitHub or HTTP will also work. Once the handler is installed on the system, enable it in the client.rb file by requiring it. After the handler is installed, it may require additional configuration. This will vary from handler to handler. If a handler is a very simple handler, it may only require the creation of a new instance. For example, if a handler named ``MyOrg::EmailMe`` is hardcoded for all of the values required to send email, a new instance is required. And then the custom handler must be associated with each of the handler types for which it will run.

For example:

.. code-block:: ruby

   require '/var/chef/handlers/email_me'         # the installation path

   email_handler = MyOrg::EmailMe.new            # a simple handler

   start_handlers << email_handler               # run at the start of the run
   report_handlers << email_handler              # run at the end of a successful run
   exception_handlers << email_handler           # run at the end of a failed run

Start Handlers
=====================================================
.. tag handler_type_start

A start handler is not loaded into a Chef Infra Client run from a recipe, but is instead listed in the client.rb file using the ``start_handlers`` attribute. The start handler must be installed on the node and be available to Chef Infra Client prior to the start of a Chef Infra Client run. Use the **chef-client** cookbook to install the start handler.

Start handlers are made available to a Chef Infra Client run in one of the following ways:

* By adding a start handler to the **chef-client** cookbook, which installs the handler on the node so that it is available to Chef Infra Client at the start of a Chef Infra Client run
* By adding the handler to one of the following settings in the node's client.rb file: ``start_handlers``

.. end_tag

Run from Recipes
-----------------------------------------------------
.. tag handler_type_start_run_from_recipe

The **chef-client** cookbook can be configured to automatically install and configure gems that are required by a start handler. For example:

.. code-block:: ruby

   node.normal['chef_client']['load_gems']['chef-reporting'] = {
     :require_name => 'chef_reporting',
     :action => :install
   }

   node.normal['chef_client']['config']['start_handlers'] = [
     {
       :class => 'Chef::Reporting::StartHandler',
       :arguments => []
     }
   ]

   include_recipe 'chef-client::config'

.. end_tag

Run from client.rb
-----------------------------------------------------
A start handler can be configured in the client.rb file by adding the following setting:

.. list-table::
   :widths: 200 300
   :header-rows: 1

   * - Setting
     - Description
   * - ``start_handlers``
     - A list of start handlers that are available to Chef Infra Client at the start of a Chef Infra Client run.

For example, the Reporting start handler adds the following code to the top of the client.rb file:

.. code-block:: ruby

   begin
     require 'chef_reporting'
     start_handlers << Chef::Reporting::StartHandler.new()
   rescue LoadError
     Chef::Log.warn 'Failed to load #{lib}. This should be resolved after a chef run.'
   end

This ensures that when a Chef Infra Client run begins the ``chef_reporting`` event handler is enabled. The ``chef_reporting`` event handler is part of a gem named ``chef-reporting``. The **chef_gem** resource is used to install this gem:

.. code-block:: ruby

   chef_gem 'chef-reporting' do
     action :install
   end

Event Handlers
=====================================================
.. tag dsl_handler_summary

Use the Handler DSL to attach a callback to an event. If the event occurs during a Chef Infra Client run, the associated callback is executed. For example:

* Sending email if a Chef Infra Client run fails
* Aggregating statistics about resources updated during a Chef Infra Client runs to StatsD

.. end_tag

on Method
-----------------------------------------------------
.. tag dsl_handler_method_on

Use the ``on`` method to associate an event type with a callback. The callback defines what steps are taken if the event occurs during a Chef Infra Client run and is defined using arbitrary Ruby code. The syntax is as follows:

.. code-block:: ruby

   Chef.event_handler do
     on :event_type do
       # some Ruby
     end
   end

where

* ``Chef.event_handler`` declares a block of code within a recipe that is processed when the named event occurs during a Chef Infra Client run
* ``on`` defines the block of code that will tell Chef Infra Client how to handle the event
* ``:event_type`` is a valid exception event type, such as ``:run_start``, ``:run_failed``, ``:converge_failed``, ``:resource_failed``, or ``:recipe_not_found``

For example:

.. code-block:: bash

   Chef.event_handler do
     on :converge_start do
       puts "Ohai! I have started a converge."
     end
   end

.. end_tag

Event Types
-----------------------------------------------------
.. tag dsl_handler_event_types

The following table describes the events that may occur during a Chef Infra Client run. Each of these events may be referenced in an ``on`` method block by declaring it as the event type.

.. list-table::
   :widths: 100 420
   :header-rows: 1

   * - Event
     - Description
   * - ``:run_start``
     - The start of a Chef Infra Client run.
   * - ``:run_started``
     - The Chef Infra Client run has started.
   * - ``:ohai_completed``
     - The Ohai run has completed.
   * - ``:skipping_registration``
     - The Chef Infra Client is not registering with the Chef Infra Server because it already has a private key or because it does not need one.
   * - ``:registration_start``
     - The Chef Infra Client is attempting to create a private key with which to register to the Chef Infra Server.
   * - ``:registration_completed``
     - The Chef Infra Client created its private key successfully.
   * - ``:registration_failed``
     - The Chef Infra Client encountered an error and was unable to register with the Chef Infra Server.
   * - ``:node_load_start``
     - The Chef Infra Client is attempting to load node data from the Chef Infra Server.
   * - ``:node_load_failed``
     - The Chef Infra Client encountered an error and was unable to load node data from the Chef Infra Server.
   * - ``:run_list_expand_failed``
     - The Chef Infra Client failed to expand the run-list.
   * - ``:node_load_completed``
     - The Chef Infra Client successfully loaded node data from the Chef Infra Server. Default and override attributes for roles have been computed, but are not yet applied.
   * - ``:policyfile_loaded``
     - The policy file was loaded.
   * - ``:cookbook_resolution_start``
     - The Chef Infra Client is attempting to pull down the cookbook collection from the Chef Infra Server.
   * - ``:cookbook_resolution_failed``
     - The Chef Infra Client failed to pull down the cookbook collection from the Chef Infra Server.
   * - ``:cookbook_resolution_complete``
     - The Chef Infra Client successfully pulled down the cookbook collection from the Chef Infra Server.
   * - ``:cookbook_clean_start``
     - The Chef Infra Client is attempting to remove unneeded cookbooks.
   * - ``:removed_cookbook_file``
     - The Chef Infra Client removed a file from a cookbook.
   * - ``:cookbook_clean_complete``
     - The Chef Infra Client is done removing cookbooks and/or cookbook files.
   * - ``:cookbook_sync_start``
     - The Chef Infra Client is attempting to synchronize cookbooks.
   * - ``:synchronized_cookbook``
     - The Chef Infra Client is attempting to synchronize the named cookbook.
   * - ``:updated_cookbook_file``
     - The Chef Infra Client updated the named file in the named cookbook.
   * - ``:cookbook_sync_failed``
     - The Chef Infra Client was unable to synchronize cookbooks.
   * - ``:cookbook_sync_complete``
     - The Chef Infra Client is finished synchronizing cookbooks.
   * - ``:library_load_start``
     - The Chef Infra Client is loading library files.
   * - ``:library_file_loaded``
     - The Chef Infra Client successfully loaded the named library file.
   * - ``:library_file_load_failed``
     - The Chef Infra Client was unable to load the named library file.
   * - ``:library_load_complete``
     - The Chef Infra Client is finished loading library files.
   * - ``:lwrp_load_start``
     - The Chef Infra Client is loading custom resources.
   * - ``:lwrp_file_loaded``
     - The Chef Infra Client successfully loaded the named custom resource.
   * - ``:lwrp_file_load_failed``
     - The Chef Infra Client was unable to load the named custom resource.
   * - ``:lwrp_load_complete``
     - The Chef Infra Client is finished loading custom resources.
   * - ``:attribute_load_start``
     - The Chef Infra Client is loading attribute files.
   * - ``:attribute_file_loaded``
     - The Chef Infra Client successfully loaded the named attribute file.
   * - ``:attribute_file_load_failed``
     - The Chef Infra Client was unable to load the named attribute file.
   * - ``:attribute_load_complete``
     - The Chef Infra Client is finished loading attribute files.
   * - ``:definition_load_start``
     - The Chef Infra Client is loading definitions.
   * - ``:definition_file_loaded``
     - The Chef Infra Client successfully loaded the named definition.
   * - ``:definition_file_load_failed``
     - The Chef Infra Client was unable to load the named definition.
   * - ``:definition_load_complete``
     - The Chef Infra Client is finished loading definitions.
   * - ``:recipe_load_start``
     - The Chef Infra Client is loading recipes.
   * - ``:recipe_file_loaded``
     - The Chef Infra Client successfully loaded the named recipe.
   * - ``:recipe_file_load_failed``
     - The Chef Infra Client was unable to load the named recipe.
   * - ``:recipe_not_found``
     - The Chef Infra Client was unable to find the named recipe.
   * - ``:recipe_load_complete``
     - The Chef Infra Client is finished loading recipes.
   * - ``:converge_start``
     - The Chef Infra Client run converge phase has started.
   * - ``:converge_complete``
     - The Chef Infra Client run converge phase is complete.
   * - ``:converge_failed``
     - The Chef Infra Client run converge phase has failed.
   * - ``:control_group_started``
     - The named control group is being processed.
   * - ``:control_example_success``
     - The named control group has been processed.
   * - ``:control_example_failure``
     - The named control group's processing has failed.
   * - ``:resource_action_start``
     - A resource action is starting.
   * - ``:resource_skipped``
     - A resource action was skipped.
   * - ``:resource_current_state_loaded``
     - A resource's current state was loaded.
   * - ``:resource_current_state_load_bypassed``
     - A resource's current state was not loaded because the resource does not support why-run mode.
   * - ``:resource_bypassed``
     - A resource action was skipped because the resource does not support why-run mode.
   * - ``:resource_update_applied``
     - A change has been made to a resource. (This event occurs for each change made to a resource.)
   * - ``:resource_failed_retriable``
     - A resource action has failed and will be retried.
   * - ``:resource_failed``
     - A resource action has failed and will not be retried.
   * - ``:resource_updated``
     - A resource requires modification.
   * - ``:resource_up_to_date``
     - A resource is already correct.
   * - ``:resource_completed``
     - All actions for the resource are complete.
   * - ``:stream_opened``
     - A stream has opened.
   * - ``:stream_closed``
     - A stream has closed.
   * - ``:stream_output``
     - A chunk of data from a single named stream.
   * - ``:handlers_start``
     - The handler processing phase of a Chef Infra Client run has started.
   * - ``:handler_executed``
     - The named handler was processed.
   * - ``:handlers_completed``
     - The handler processing phase of a Chef Infra Client run is complete.
   * - ``:provider_requirement_failed``
     - An assertion declared by a provider has failed.
   * - ``:whyrun_assumption``
     - An assertion declared by a provider has failed, but execution is allowed to continue because the Chef Infra Client is running in why-run mode.
   * - ``:run_completed``
     - The Chef Infra Client run has completed.
   * - ``:run_failed``
     - The Chef Infra Client run has failed.
   * - ``:attribute_changed``
     - Prints out all the attribute changes in cookbooks or sets a policy that override attributes should never be used.

.. end_tag

Examples
-----------------------------------------------------
The following examples show ways to use the Handler DSL.

Send Email
+++++++++++++++++++++++++++++++++++++++++++++++++++++
.. tag dsl_handler_slide_send_email

Use the ``on`` method to create an event handler that sends email when a Chef Infra Client run fails. This will require:

* A way to tell Chef Infra Client how to send email
* An event handler that describes what to do when the ``:run_failed`` event is triggered
* A way to trigger the exception and test the behavior of the event handler

.. end_tag

**Define How Email is Sent**

.. tag dsl_handler_slide_send_email_library

Use a library to define the code that sends email when a Chef Infra Client run fails. Name the file ``helper.rb`` and add it to a cookbook's ``/libraries`` directory:

.. code-block:: ruby

   require 'net/smtp'

   module HandlerSendEmail
     class Helper

       def send_email_on_run_failure(node_name)

         message = "From: Chef <chef@chef.io>\n"
         message << "To: Grant <grantmc@chef.io>\n"
         message << "Subject: Chef run failed\n"
         message << "Date: #{Time.now.rfc2822}\n\n"
         message << "Chef run failed on #{node_name}\n"
         Net::SMTP.start('localhost', 25) do |smtp|
           smtp.send_message message, 'chef@chef.io', 'grantmc@chef.io'
         end
       end
     end
   end

.. end_tag

**Add the Handler**

.. tag dsl_handler_slide_send_email_handler

Invoke the library helper in a recipe:

.. code-block:: ruby

   Chef.event_handler do
     on :run_failed do
       HandlerSendEmail::Helper.new.send_email_on_run_failure(
         Chef.run_context.node.name
       )
     end
   end

* Use ``Chef.event_handler`` to define the event handler
* Use the ``on`` method to specify the event type

Within the ``on`` block, tell Chef Infra Client how to handle the event when it's triggered.

.. end_tag

**Test the Handler**

.. tag dsl_handler_slide_send_email_test

Use the following code block to trigger the exception and have the Chef Infra Client send email to the specified email address:

.. code-block:: ruby

   ruby_block 'fail the run' do
     block do
       fail 'deliberately fail the run'
     end
   end

.. end_tag

etcd Locks
+++++++++++++++++++++++++++++++++++++++++++++++++++++
.. tag dsl_handler_example_etcd_lock

The following example shows how to prevent concurrent Chef Infra Client runs from both holding a lock on etcd:

.. code-block:: ruby

   lock_key = "#{node.chef_environment}/#{node.name}"

   Chef.event_handler do
     on :converge_start do |run_context|
       Etcd.lock_acquire(lock_key)
     end
   end

   Chef.event_handler do
     on :converge_complete do
       Etcd.lock_release(lock_key)
     end
   end

.. end_tag

HipChat Notifications
+++++++++++++++++++++++++++++++++++++++++++++++++++++
.. tag dsl_handler_example_hipchat

Event messages can be sent to a team communication tool like HipChat. For example, if a Chef Infra Client run fails:

.. code-block:: ruby

   Chef.event_handler do
     on :run_failed do |exception|
       hipchat_notify exception.message
     end
   end

or send an alert on a configuration change:

.. code-block:: ruby

   Chef.event_handler do
     on :resource_updated do |resource, action|
       if resource.to_s == 'template[/etc/nginx/nginx.conf]'
         Helper.hipchat_message("#{resource} was updated by chef")
       end
     end
   end

.. end_tag

Handlers and Cookbooks
=====================================================
The following cookbooks can be used to load handlers during a Chef Infra Client run.

chef_handler
-----------------------------------------------------
Exception and report handlers can be distributed using the **chef_handler** cookbook. This cookbook is authored and maintained by Chef and exposes a custom resource that can be used to enable custom handlers from within recipes and to include product-specific handlers from cookbooks. The **chef_handler** cookbook can be accessed here: https://github.com/chef-cookbooks/chef_handler. See the ``README.md`` for additional information.

Chef Infra Client
-----------------------------------------------------
Start handlers can be distributed using the **chef-client** cookbook, which will install the handler on the target node during the initial configuration of the node. This ensures that the start handler is always present on the node so that it is available to Chef Infra Client at the start of every run.

Custom Handlers
=====================================================
.. tag handler_custom

A custom handler can be created to support any situation. The easiest way to build a custom handler:

#. Download the **chef_handler** cookbook
#. Create a custom handler
#. Write a recipe using the **chef_handler** resource
#. Add that recipe to a node's run-list, often as the first recipe in that run-list

.. end_tag

Syntax
-----------------------------------------------------
.. tag handler_custom_syntax

The syntax for a handler can vary, depending on what the the situations the handler is being asked to track, the type of handler being used, and so on. All custom exception and report handlers are defined using Ruby and must be a subclass of the ``Chef::Handler`` class.

.. code-block:: ruby

   require 'chef/log'

   module ModuleName
     class HandlerName < Chef::Handler
       def report
         # Ruby code goes here
       end
     end
   end

where:

* ``require`` ensures that the logging functionality of Chef Infra Client is available to the handler
* ``ModuleName`` is the name of the module as it exists within the ``Chef`` library
* ``HandlerName`` is the name of the handler as it is used in a recipe
* ``report`` is an interface that is used to define the custom handler

For example, the following shows a custom handler that sends an email that contains the exception data when a Chef Infra Client run fails:

.. code-block:: ruby

   require 'net/smtp'

   module OrgName
     class SendEmail < Chef::Handler
       def report
         if run_status.failed? then
           message  = "From: sender_name <sender@example.com>\n"
           message << "To: recipient_address <recipient@example.com>\n"
           message << "Subject: chef-client Run Failed\n"
           message << "Date: #{Time.now.rfc2822}\n\n"
           message << "Chef run failed on #{node.name}\n"
           message << "#{run_status.formatted_exception}\n"
           message << Array(backtrace).join('\n')
           Net::SMTP.start('your.smtp.server', 25) do |smtp|
             smtp.send_message message, 'sender@example', 'recipient@example'
           end
         end
       end
     end
   end

and then is used in a recipe like:

.. code-block:: ruby

   send_email 'blah' do
     # recipe code
   end

.. end_tag

report Interface
-----------------------------------------------------
.. tag handler_custom_interface_report

The ``report`` interface is used to define how a handler will behave and is a required part of any custom handler. The syntax for the ``report`` interface is as follows:

.. code-block:: ruby

   def report
     # Ruby code
   end

The Ruby code used to define a custom handler will vary significantly from handler to handler. Chef Infra Client includes two default handlers: ``error_report`` and ``json_file``. Their use of the ``report`` interface is shown below.

The `error_report <https://github.com/chef/chef/blob/master/lib/chef/handler/error_report.rb>`_ handler:

.. code-block:: ruby

   require 'chef/handler'
   require 'chef/resource/directory'

   class Chef
     class Handler
       class ErrorReport < ::Chef::Handler
         def report
           Chef::FileCache.store('failed-run-data.json', Chef::JSONCompat.to_json_pretty(data), 0640)
           Chef::Log.fatal("Saving node information to #{Chef::FileCache.load('failed-run-data.json', false)}")
         end
       end
    end
   end

The `json_file <https://github.com/chef/chef/blob/master/lib/chef/handler/json_file.rb>`_ handler:

.. code-block:: ruby

   require 'chef/handler'
   require 'chef/resource/directory'

   class Chef
     class Handler
       class JsonFile < ::Chef::Handler
         attr_reader :config
         def initialize(config={})
           @config = config
           @config[:path] ||= '/var/chef/reports'
           @config
         end
         def report
           if exception
             Chef::Log.error('Creating JSON exception report')
           else
             Chef::Log.info('Creating JSON run report')
           end
           build_report_dir
           savetime = Time.now.strftime('%Y%m%d%H%M%S')
           File.open(File.join(config[:path], 'chef-run-report-#{savetime}.json'), 'w') do |file|
             run_data = data
             run_data[:start_time] = run_data[:start_time].to_s
             run_data[:end_time] = run_data[:end_time].to_s
             file.puts Chef::JSONCompat.to_json_pretty(run_data)
           end
         end
         def build_report_dir
           unless File.exist?(config[:path])
             FileUtils.mkdir_p(config[:path])
             File.chmod(00700, config[:path])
           end
         end
       end
     end
   end

.. end_tag

Optional Interfaces
-----------------------------------------------------
The following interfaces may be used in a handler in the same way as the ``report`` interface to override the default handler behavior in Chef Infra Client. That said, the following interfaces are not typically used in a handler and, for the most part, are completely unnecessary for a handler to work properly and/or as desired.

data
+++++++++++++++++++++++++++++++++++++++++++++++++++++
.. tag handler_custom_interface_data

The ``data`` method is used to return the Hash representation of the ``run_status`` object. For example:

.. code-block:: ruby

   def data
     @run_status.to_hash
   end

.. end_tag

run_report_safely
+++++++++++++++++++++++++++++++++++++++++++++++++++++
.. tag handler_custom_interface_run_report_safely

The ``run_report_safely`` method is used to run the report handler, rescuing and logging errors that may arise as the handler runs and ensuring that all handlers get a chance to run during a Chef Infra Client run (even if some handlers fail during that run). In general, this method should never be used as an interface in a custom handler unless this default behavior simply must be overridden.

.. code-block:: ruby

   def run_report_safely(run_status)
     run_report_unsafe(run_status)
   rescue Exception => e
     Chef::Log.error('Report handler #{self.class.name} raised #{e.inspect}')
     Array(e.backtrace).each { |line| Chef::Log.error(line) }
   ensure
     @run_status = nil
   end

.. end_tag

run_report_unsafe
+++++++++++++++++++++++++++++++++++++++++++++++++++++
.. tag handler_custom_interface_run_report_unsafe

The ``run_report_unsafe`` method is used to run the report handler without any error handling. This method should never be used directly in any handler, except during testing of that handler. For example:

.. code-block:: ruby

   def run_report_unsafe(run_status)
     @run_status = run_status
     report
   end

.. end_tag

run_status Object
-----------------------------------------------------
.. tag handler_custom_object_run_status

The ``run_status`` object is initialized by Chef Infra Client before the ``report`` interface is run for any handler. The ``run_status`` object keeps track of the status of a Chef Infra Client run and will contain some (or all) of the following properties:

.. list-table::
   :widths: 200 300
   :header-rows: 1

   * - Property
     - Description
   * - ``all_resources``
     - A list of all resources that are included in the ``resource_collection`` property for the current Chef Infra Client run.
   * - ``backtrace``
     - A backtrace associated with the uncaught exception data that caused a Chef Infra Client run to fail, if present; ``nil`` for a successful Chef Infra Client run.
   * - ``elapsed_time``
     - The amount of time between the start (``start_time``) and end (``end_time``) of a Chef Infra Client run.
   * - ``end_time``
     - The time at which a Chef Infra Client run ended.
   * - ``exception``
     - The uncaught exception data which caused a Chef Infra Client run to fail; ``nil`` for a successful Chef Infra Client run.
   * - ``failed?``
     - Show that a Chef Infra Client run has failed when uncaught exceptions were raised during a Chef Infra Client run. An exception handler runs when the ``failed?`` indicator is ``true``.
   * - ``node``
     - The node on which a Chef Infra Client run occurred.
   * - ``run_context``
     - An instance of the ``Chef::RunContext`` object; used by Chef Infra Client to track the context of the run; provides access to the ``cookbook_collection``, ``resource_collection``, and ``definitions`` properties.
   * - ``start_time``
     - The time at which a Chef Infra Client run started.
   * - ``success?``
     - Show that a Chef Infra Client run succeeded when uncaught exceptions were not raised during a Chef Infra Client run. A report handler runs when the ``success?`` indicator is ``true``.
   * - ``updated_resources``
     - A list of resources that were marked as updated as a result of a Chef Infra Client run.

.. note:: These properties are not always available. For example, a start handler runs at the beginning of Chef Infra Client run, which means that properties like ``end_time`` and ``elapsed_time`` are still unknown and will be unavailable to the ``run_status`` object.

.. end_tag

Examples
=====================================================
The following sections show examples of handlers.

Cookbook Versions
-----------------------------------------------------
.. tag handler_custom_example_cookbook_versions

Community member ``juliandunn`` created a custom `report handler that logs all of the cookbooks and cookbook versions <https://github.com/juliandunn/cookbook_versions_handler>`_ that were used during a Chef Infra Client run, and then reports after the run is complete. This handler requires the **chef_handler** resource (which is available from the **chef_handler** cookbook).

.. end_tag

cookbook_versions.rb
+++++++++++++++++++++++++++++++++++++++++++++++++++++
.. tag handler_custom_example_cookbook_versions_handler

The following custom handler defines how cookbooks and cookbook versions that are used during a Chef Infra Client run will be compiled into a report using the ``Chef::Log`` class in Chef Infra Client:

.. code-block:: ruby

   require 'chef/log'

   module Opscode
     class CookbookVersionsHandler < Chef::Handler

       def report
         cookbooks = run_context.cookbook_collection
         Chef::Log.info('Cookbooks and versions run: #{cookbooks.keys.map {|x| cookbooks[x].name.to_s + ' ' + cookbooks[x].version} }')
       end
     end
   end

.. end_tag

default.rb
+++++++++++++++++++++++++++++++++++++++++++++++++++++
.. tag handler_custom_example_cookbook_versions_recipe

The following recipe is added to the run-list for every node on which a list of cookbooks and versions will be generated as report output after every Chef Infra Client run.

.. code-block:: ruby

   include_recipe 'chef_handler'

   cookbook_file "#{node['chef_handler']['handler_path']}/cookbook_versions.rb" do
     source 'cookbook_versions.rb'
     owner 'root'
     group 'root'
     mode '0755'
     action :create
   end

   chef_handler 'Opscode::CookbookVersionsHandler' do
     source "#{node['chef_handler']['handler_path']}/cookbook_versions.rb"
     supports :report => true
     action :enable
   end

This recipe will generate report output similar to the following:

.. code-block:: ruby

   [2013-11-26T03:11:06+00:00] INFO: Chef Run complete in 0.300029878 seconds
   [2013-11-26T03:11:06+00:00] INFO: Running report handlers
   [2013-11-26T03:11:06+00:00] INFO: Cookbooks and versions run: ["chef_handler 1.1.4", "cookbook_versions_handler 1.0.0"]
   [2013-11-26T03:11:06+00:00] INFO: Report handlers complete

.. end_tag

Reporting
-----------------------------------------------------
Start handler functionality was added when Chef started building add-ons for the Chef Infra Server. The Reporting add-on is designed to create reporting data based on a Chef Infra Client run. And since Reporting needs to be able to collect data for the entire Chef Infra Client run, Reporting needs to be enabled before anything else happens at the start of a Chef Infra Client run.

.. note:: The start handler used by the Reporting add-on for the Chef Infra Server is always installed using the **chef-client** cookbook.

start_handler.rb
+++++++++++++++++++++++++++++++++++++++++++++++++++++
The following code shows the start handler used by the Reporting add-in for the Chef Infra Server:

.. code-block:: ruby

   require 'chef/handler'
   require 'chef/rest'
   require 'chef/version_constraint'

   class Chef
     class Reporting
       class StartHandler < ::Chef::Handler

         attr_reader :config

         def initialize(config={})
           @config = config
         end

         def report
           version_checker = Chef::VersionConstraint.new('< 11.6.0')
           if version_checker.include?(Chef::VERSION)
             Chef::Log.info('Enabling backported resource reporting Handler')
             rest = Chef::REST.new(Chef::Config[:chef_server_url], @run_status.node.name, Chef::Config[:client_key])
             resource_reporter = Chef::Reporting::ResourceReporter.new(rest)
             @run_status.events.register(resource_reporter)

             resource_reporter.run_started(@run_status)
           else
            Chef::Log.debug('Chef Version already has new Resource Reporter - skipping startup of backport version')
           end
         end
       end
     end
   end

json_file Handler
-----------------------------------------------------
.. tag handler_custom_example_json_file

The `json_file <https://github.com/chef/chef/blob/master/lib/chef/handler/json_file.rb>`_ handler is available from the **chef_handler** cookbook and can be used with exceptions and reports. It serializes run status data to a JSON file. This handler may be enabled in one of the following ways.

By adding the following lines of Ruby code to either the client.rb file or the solo.rb file, depending on how Chef Infra Client is being run:

.. code-block:: ruby

   require 'chef/handler/json_file'
   report_handlers << Chef::Handler::JsonFile.new(:path => '/var/chef/reports')
   exception_handlers << Chef::Handler::JsonFile.new(:path => '/var/chef/reports')

By using the **chef_handler** resource in a recipe, similar to the following:

.. code-block:: ruby

   chef_handler 'Chef::Handler::JsonFile' do
     source 'chef/handler/json_file'
     arguments :path => '/var/chef/reports'
     action :enable
   end

After it has run, the run status data can be loaded and inspected via Interactive Ruby (IRb):

.. code-block:: ruby

   irb(main):002:0> require 'json' => true
   irb(main):003:0> require 'chef' => true
   irb(main):004:0> r = JSON.parse(IO.read('/var/chef/reports/chef-run-report-20110322060731.json')) => ... output truncated
   irb(main):005:0> r.keys => ['end_time', 'node', 'updated_resources', 'exception', 'all_resources', 'success', 'elapsed_time', 'start_time', 'backtrace']
   irb(main):006:0> r['elapsed_time'] => 0.00246

.. end_tag

error_report Handler
-----------------------------------------------------
.. tag handler_custom_example_error_report

The `error_report <https://github.com/chef/chef/blob/master/lib/chef/handler/error_report.rb>`_ handler is built into Chef Infra Client and can be used for both exceptions and reports. It serializes error report data to a JSON file. This handler may be enabled in one of the following ways.

By adding the following lines of Ruby code to either the client.rb file or the solo.rb file, depending on how Chef Infra Client is being run:

.. code-block:: ruby

   require 'chef/handler/error_report'
   report_handlers << Chef::Handler::ErrorReport.new()
   exception_handlers << Chef::Handler::ErrorReport.new()

By using the `chef_handler </resource_chef_handler.html>`__ resource in a recipe, similar to the following:

.. code-block:: ruby

   chef_handler 'Chef::Handler::ErrorReport' do
     source 'chef/handler/error_report'
     action :enable
   end

.. end_tag

Community Handlers
-----------------------------------------------------
.. tag handler_community_handlers

The following open source handlers are available from the Chef community:

.. list-table::
   :widths: 60 420
   :header-rows: 1

   * - Handler
     - Description
   * - `Airbrake <https://github.com/timops/ohai-plugins/blob/master/win32_svc.rb>`_
     - A handler that sends exceptions (only) to Airbrake, an application that collects data and aggregates it for review.
   * - `Asynchronous Resources <https://github.com/rottenbytes/chef/tree/master/async_handler>`_
     - A handler that asynchronously pushes exception and report handler data to a STOMP queue, from which data can be processed into data storage.
   * - `Campfire <https://github.com/ampledata/chef-handler-campfire>`_
     - A handler that collects exception and report handler data and reports it to Campfire, a web-based group chat tool.
   * - `Datadog <https://github.com/DataDog/chef-handler-datadog>`_
     - A handler that collects Chef Infra Client stats and sends them into a Datadog newsfeed.
   * - `Flowdock <https://github.com/mmarschall/chef-handler-flowdock>`_
     - A handler that collects exception and report handler data and sends it to users via the Flowdock API..
   * - `Graphite <https://github.com/imeyer/chef-handler-graphite/wiki>`_
     - A handler that collects exception and report handler data and reports it to Graphite, a graphic rendering application.
   * - `Graylog2 GELF <https://github.com/jellybob/chef-gelf/>`_
     - A handler that provides exception and report handler status (including changes) to a Graylog2 server, so that the data can be viewed using Graylog Extended Log Format (GELF).
   * - `Growl <https://rubygems.org/gems/chef-handler-growl>`_
     - A handler that collects exception and report handler data and then sends it as a Growl notification.
   * - `HipChat <https://github.com/mojotech/hipchat/blob/master/lib/hipchat/chef.rb>`_
     - A handler that collects exception handler data and sends it to HipChat, a hosted private chat service for companies and teams.
   * - `IRC Snitch <https://rubygems.org/gems/chef-irc-snitch>`_
     - A handler that notifies administrators (via Internet Relay Chat (IRC)) when a Chef Infra Client run fails.
   * - `Journald <https://github.com/marktheunissen/chef-handler-journald>`_
     - A handler that logs an entry to the systemd journal with the Chef Infra Client run status, exception details, configurable priority, and custom details.
   * - `net/http <https://github.com/b1-systems/chef-handler-httpapi/>`_
     - A handler that reports the status of a Chef run to any API via net/HTTP.
   * - `Simple Email <https://rubygems.org/gems/chef-handler-mail>`_
     - A handler that collects exception and report handler data and then uses pony to send email reports that are based on Erubis templates.
   * - `SendGrid Mail Handler <https://github.com/sendgrid-ops/chef-sendgrid_mail_handler>`_
     - A chef handler that collects exception and report handler data and then uses SendGrid Ruby gem to send email reports that are based on Erubis templates.
   * - `SNS <http://onddo.github.io/chef-handler-sns/>`_
     - A handler that notifies exception and report handler data and sends it to a SNS topic.
   * - `Slack <https://github.com/rackspace-cookbooks/chef-slack_handler>`_
     - A handler to send Chef Infra Client run notifications to a Slack channel.
   * - `Splunk Storm <http://ampledata.org/splunk_storm_chef_handler.html>`_
     - A handler that supports exceptions and reports for Splunk Storm.
   * - `Syslog <https://github.com/jblaine/syslog_handler>`_
     - A handler that logs basic essential information, such as about the success or failure of a Chef Infra Client run.
   * - `Updated Resources <https://rubygems.org/gems/chef-handler-updated-resources>`_
     - A handler that provides a simple way to display resources that were updated during a Chef Infra Client run.
   * - `ZooKeeper <http://onddo.github.io/chef-handler-zookeeper/>`_
     - A Chef report handler to send Chef run notifications to ZooKeeper.

.. end_tag
