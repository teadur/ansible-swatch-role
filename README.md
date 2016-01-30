Ansible Swatch Role
=========

An Ansible role to install Swatch: The Simple Log Watcher with a handy Slack integration & monit files

Requirements
------------

Tested on Ubuntu Trusty and Precise. Uses apt to install things so it won't work on RedHat etc as it is written.

Role Variables
--------------

This role is designed to watch 4 different log files (by default).

```
/var/log/docker.log
/var/log/syslog
/var/log/auth.log
/var/log/apache/error.log
```

You'll also need to set {{ swatch_email_to }} somewhere to have email delivered.

This role will also post to Slack from swatch using curl in a shell script. This uses the following vars with examples:

```
swatch_slack_channel: "swatch"
swatch_slack_username: "swatchbot"
swatch_slack_error_text: " Swatch error from : _'$HOSTNAME'_ : `\'\"\'$ERROR\'\"\'` "
swatch_slack_icon: ":bangbang:"
swatch_slack_webhook_url: "https://hooks.slack.com/services/bloop/dodo/something"
```

that `swatch_slack_error_text:` can be a bit tricky with all the quotes and stuff.

Those vars can be used to build a shell script like this:

```
#!/bin/bash
ERROR=$1
curl -X POST --data-urlencode 'payload={"channel": "{{ swatch_slack_channel }}", "username": "{{ swatch_slack_username }}", "text": "{{ swatch_slack_error_text }}", "icon_emoji": "{{ swatch_slack_icon }}"}' \
 {{ swatch_slack_webhook_url }}

```

That shell script ends up in `/usr/local/bin/slack-swatch.sh`

You'll also want to add the var picked up to make all the files:

```
monit_swatch:
   - syslog
   - auth.log
   - docker.log
```

Example Playbook
----------------

```ansible-playbook -i [some inventory file] -e ansible_ssh_port=[port] -u user -K ./swatch.yml --limit=[server] --tags=swatch --extra-vars '{"swatch_logs": ["syslog", "auth.log"]}' --extra-vars "swatch_email_to=none@example.com" ```

We only have some servers that run Apache and use the SoftLayer Dynamic Inventory script to handle inventory. All our servers are tagged at SoftLayer so we can use the `--limit` in Ansible to run against many servers at once, here's how I only install the apache swatch on our Apache servers:

```ansible-playbook -i softlayer.py -e ansible_ssh_port=22 -u user -K ./swatch.yml --limit=apache --tags=swatch --extra-vars '{"swatch_logs": ["syslog", "auth.log", "/apache/error.log"]}' --extra-vars "swatch_email_to=none@example.com"```

here's how I only install Swatch on our servers runing Docker:

```ansible-playbook -i softlayer.py -e ansible_ssh_port=22 -u user -K ./swatch.yml --limit=docker --tags=swatch --extra-vars '{"swatch_logs": ["syslog", "auth.log", "docker.log"]}' --extra-vars "swatch_email_to=none@example.com"```

You do need the `/apache/error.log` to be set more like a path like that. Have a look at the bash in swatchd.j2, it does a little replace `${i/\//-}` to be able to use that as
both a path and a file name. 

Author Information
------------------

- Blake Carver
