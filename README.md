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

To start the Swatch for each type of log & fire up all four do 
```--extra-vars '{"logs": ["syslog", "auth.log", "docker.log", "/apache/error.log"]}'```


You'll also need to set {{ email_to }} somewhere to have email delivered.

This role will also post to Slack from swatch using curl in a shell script. This uses the following vars:
```
swatch_channel: "#swatch"
swatch_username: "webhookbot"
swatch_error_text: "Swatch error from server '$HOSTNAME': '"'$ERROR'"'" " 
swatch_icon: ":bug:"  
swatch_webhook_url: "https://hooks.slack.com/services/EGGHOP/PUMPkin/L33t"
```

Those vars can be used to build a shell script like this:
```
#!/bin/bash
ERROR=$1
curl -X POST --data-urlencode 'payload={"channel": "{{ swatch_channel }}", "username": "{{ swatch_username }}", "text": "{{ swatch_error_text }}", "icon_emoji": "{{ swatch_icon }}"}' \
 {{ swatch_webhook_url }}

```
That shell script ends up in `/usr/local/bin/slack.sh`

Example Playbook
----------------

```ansible-playbook -i [some inventory file] -e ansible_ssh_port=[port] -u user -K ./swatch.yml --limit=[server] --tags=swatch --extra-vars '{"logs": ["syslog", "auth.log"]}' --extra-vars "email_to=none@example.com" ```

We only have some servers that run Apache and use the SoftLayer Dynamic Inventory script to handle inventory. All our servers are tagged at SoftLayer so we can use the `--limit` in Ansible to run against many servers at once, here's how I only install the apache swatch on our Apache servers:
```ansible-playbook -i softlayer.py -e ansible_ssh_port=22 -u user -K ./swatch.yml --limit=apache --tags=swatch --extra-vars '{"logs": ["syslog", "auth.log", "/apache/error.log"]}' --extra-vars "email_to=none@example.com"```

here's how I only install Swatch on our servers runing Docker:
```ansible-playbook -i softlayer.py -e ansible_ssh_port=22 -u user -K ./swatch.yml --limit=docker --tags=swatch --extra-vars '{"logs": ["syslog", "auth.log", "docker.log"]}' --extra-vars "email_to=none@example.com"```


You do need the `/apache/error.log` to be set more like a path like that. Have a look at the bash in swatchd.j2, it does a little replace `${i/\//-}` to be able to use that as
both a path and a file name. 

Author Information
------------------

- Blake Carver
