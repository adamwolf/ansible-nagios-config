Ansible-Nagios-Config
====================

Before going any further, this role is designed to be run on servers that
have been provisioned using the npm [Nagios Role](http://github.com/npm/ansible-nagios).
Provision a new server using this role.

Notes
----

* I suspect that the directive to bring in cfg_dir doesn't work fully.
* I set up a directory for SSH ControlMaster.  You can use something like the following:

```
nagios_hosts
  - name: 'echo'
    address: '192.168.17.21'
    linux: true
    checks:
      - {description: 'Load', command: "check_load_by_ssh!3.0,3.0,3.0!5.0,5.0,5.0"}

nagios_commands:
  - {name: 'check_load_by_ssh', command: "$USER1$/check_by_ssh -o ControlMaster=auto -o ControlPath=/usr/local/nagios/.ssh/controlmaster/$HOSTNAME$ -o ControlPersist=yes -H $HOSTADDRESS$ -n $HOSTNAME$ -C '/usr/lib/nagios/plugins/check_load -w $ARG1$ -c $ARG2$'"}
```


What is This?
------------

Nagios is a pain to configure, Ansible-Nagios-Config alleviates this by:

* providing sane default settings for hosts/services.
* automating the generation of configuration, through a simple DSL.

Usage
-----

* Install [librarian-ansible](https://github.com/bcoe/librarian-ansible) a tool for
managing your third-party Ansible dependencies.
* Provision a new Nagios server using [ansible-nagios](https://github.com/npm/ansible-nagios).
* Set the *nagios_host_groups*, *nagios_commands*, and *nagios_hosts* variables described
in the next section of this document.
* Create a playbook that references the *ansible-nagios-config* role, and use it to deploy
your configuration.

nagios_hosts
------------

Used to describe the hosts you would like to monitor with Nagios:

* **name:** the name of the host, used to reference it in *host_groups.*
* **address:** the actual address of the host.
* *other variables:* other variables represent *host_groups* that the host
belongs to, e.g., `www`, `aws`.

```yaml
nagios_hosts:
  - {name: 'www', address: 'www.example.com', www: true, aws: true}
  - {name: 'redis', address: 'redis.example.com', redis: true, aws: true}
```

nagios_groups
-------------

Used to organize your hosts into types of servers, e.g., web, redis, MySQL, and
to describe the checks that should be applied to these groups.

```yaml
nagios_host_groups:
  - name: 'www'
    alias: 'Web Servers'
    checks:
      - {command: 'check_http_npmjs_org', description: 'Check HTTP'}
      - {command: 'check_nrpe2!check_disk_nrpe', description: 'Disk Space Left'}
  - name: 'aws'
    alias: 'AWS Servers'
  - name: 'redis'
    alias: 'Redis Server'
    checks:
      - {command: 'check_nrpe2!check_disk_nrpe', description: 'Disk Space Left'}
```

nagios_commands
---------------

Describe custom commands that can be used to perform host checks:

```yaml
nagios_commands:
  - {name: 'check_http_npmjs_org', command: '$USER1$/check_http -H npmjs.org -I $HOSTADDRESS$ $ARG1$'}
  - {name: 'check_nrpe2', command: '$USER1$/check_nrpe -H $HOSTADDRESS$ -c $ARG1$'}
```

nagios_contacts
---------------

Describe custom contacts

```yaml
nagios_contacts:
  - {name: 'The Contact Name', shortname: 'thecontactshortname', alias: 'The Contact Alias", email: "the@contactemail.com"}
```

nagios_contactgroups
--------------------

Describe the contact groups

```yaml
nagios_contactgroups:
  - {name: 'contactgroupname', alias: 'The Contact Group Name', members: 'members,of,the,contactgroup"}
```

Enabling Slack
--------------
Set the variable nagios_enable_slack_notifications to True.  Set nagios_slack_domain to the domain, and set nagios_slack_token to the appropriate token. You almost
certainly want to put the token in an Ansible vault.  Set nagios_slack_service_channel to the channel you want service alerts posted to, and nagios_slack_host_channel
to the channel you want host alerts posted to.

Then, set it up in your contacts and contactgroup.  One way is like this:
```
nagios_contacts:
  - {name: 'nagiosadmin', shortname: 'nagiosadmin', alias: 'Nagios Admin', email: "whatever@example.com"}

nagios_contactgroups:
  - {name: 'admins', alias: 'Nagios Admins', members: 'slack,nagiosadmin'}
```

