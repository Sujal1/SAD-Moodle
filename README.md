SAD-Moodle
==========


How to Install

make sure you have apache, mysql and php running. For windows you can use Xampp or Wampp, for Ubuntu users just install apache-php5-mysql 

Download all files and import moodle.sql, modify config.php accordingly.



Configuring your Ubuntu
=========================

<code>sudo apt-get install php5-mysql apache2 libapache2-mod-php5</code>






Configuring Apache
====================

it's better to point your server to which your cloned project is.


/etc/apache2/sites-available/000-default.conf
<pre><code>DocumentRoot /home/rey/Application/SAD-Moodle</code></pre>
Add the following just below the DocumentRoot

<pre><code>&lt;Directory /home/rey/Application/SAD-Moodle&gt;
        Options Indexes FollowSymLinks
        AllowOverride None
        Require all granted
&lt;/Directory&gt;</code></pre>

Of course you need to change /home/rey/Application/SAD-Moodle to where your project is located.

  
  
How to add a new function
=========================
Adding a new function



Read more here http://docs.moodle.org/dev/Adding_a_web_service_to_a_plugin#File_structure


Insert in Database -> mdl_external_services_functions  (We need to do this if we have the plugin already installed)

externalserviceId ->3
functionname -> local_wstemplate_get_course
 
  
table mdl_external_functions
name:        local_wstemplate_get_course
classname:   local_wstemplate_external
methodname:  get_course
classpath:   local/wstemplate/externallib.php
component:   local_wstemplate

/local/wstemplate/db/services.php  (This will be use at the initial Install)


if everything is added then you will be able to see the new function in Site Administration->Plugins->Webservices->External Services then edit our Plugin then click show functions

Add one more item in array

-------
<pre><code>
$functions = array(
        'local_wstemplate_hello_world' => array(
                'classname'   => 'local_wstemplate_external',
                'methodname'  => 'hello_world',
                'classpath'   => 'local/wstemplate/externallib.php',
                'description' => 'Return Hello World FIRSTNAME. Can change the text (Hello World) sending a new text as parameter',
                'type'        => 'read',
        ),
		
		 'local_wstemplate_quick_test' => array(
                'classname'   => 'local_wstemplate_external',
                'methodname'  => 'get_child',
                'classpath'   => 'local/wstemplate/externallib.php',
                'description' => 'Return Hello World FIRSTNAME. Can change the text (Hello World) sending a new text as parameter',
                'type'        => 'read',
        )
);
</code></pre>

/local/wstemplate/externallib.php


add function call around 3 of them  like the one below

<pre><code>
 public static function get_course_parameters() {
        return new external_function_parameters(
                array('welcomemessage' => new external_value(PARAM_TEXT, 'The welcome message. By default it is "Hello worldx,"', VALUE_DEFAULT, 'Hello worldx, '))
        );
    }
		
	
	public static function get_course($welcomemessage = 'Hello world, ') {
        global $USER, $DB;
 

		$usercontexts = $DB->get_records_sql("SELECT c.instanceid, c.instanceid, u.firstname, u.lastname
                                                    FROM {role_assignments} ra, {context} c, {user} u
                                                   WHERE ra.userid = ?
                                                         AND ra.contextid = c.id
                                                         AND c.instanceid = u.id
                                                         AND c.contextlevel = ".CONTEXT_USER, array($USER->id));		
			
		$result = array();
		$count = 0;
		 foreach ($usercontexts as $usercontext) {
		        
				$result[$count]['id']       = $usercontext->instanceid;
			    $result[$count]['fullname'] =fullname($usercontext);               
				$count++;
            }
			
        return $result;;
    }
	
	 public static function get_course_returns() {
        return new external_value(PARAM_TEXT, 'The welcome message + user first name');
    }
 

}
</code></pre>
