Ansible Swatch Role
=========

An Ansible role to install Swatch: The Simple Log Watcher

Requirements
------------

Tested on Ubuntu Trusty and Precise. Uses apt to install things so it won't work on RedHat etc as it is written.

Role Variables
--------------

This role is designed to watch 4 different log files.

```
/var/log/docker.log
/var/log/syslog
/var/log/auth.log
/var/log/apache/error.log
```

Which logs to install is sent via the cli extra-vars like

``` --extra-vars '{"logs": ["syslog", "auth.log"]}'```

Those extra-vars are used by templates/swatchd.j2 where `LOGS="{{ logs|join(' ') }}"` set up the startup script to watch logs in or below /var/log.

To start the Swatch for each type of log & fire up all four do `--extra-vars '{"logs": ["syslog", "auth.log", "docker.log", "/apache/error.log"]}'`

You do need the `/apache/error.log` to be set more like a path like that. Have a look at the bash in swatchd.j2, it does a little replace `${i/\//-}` to be able to use that as
both a path and a file name. 

You'll also need to set {{ email_to }} somewhere to have email delivered.

Example Playbook
----------------

```ansible-playbook -i [some inventory file] -e ansible_ssh_port=[port] -u user -K ./swatch.yml --limit=[server] --tags=swatch --extra-vars '{"logs": ["syslog", "auth.log"]}'````

Author Information
------------------

- Blake Carver
