Ansible Swatch Role
=========

An Ansible role to install Swatch: The Simple Log Watcher with a handy Slack integration & monit files

Requirements
------------

Tested on Ubuntu Trusty and Precise. Uses apt to install things so it won't work on RedHat etc as it is written.

Role Variables
--------------

This role is designed to watch 4 different log files:

```
/var/log/docker.log
/var/log/syslog
/var/log/auth.log
/var/log/apache/error.log
```

Which files get watched gets defined by the `monit_swatch` in your vars.

You'll also need to set {{ swatch_email_to }} somewhere to have email delivered.

This role will also post to Slack from swatch using curl in a shell script. This uses the following vars with examples:

```
swatch_email_to: "none@example.com"
slack_swatch_channel: "swatch"
slack_swatch_username: "swatchbot"
slack_swatch_error_text: " Swatch error from : _'$HOSTNAME'_ : `\'\"\'$ERROR\'\"\'` "
slack_swatch_icon: ":bangbang:" 
slack_swatch_webhook_url: "https://hooks.slack.com/services/ef2wqft/efw2e4/whateverelseitlookslikehere"
swatch_inits:
   - syslog
   - auth.log
   - docker.log
swatch_confs:
   - syslog
   - auth.log
   - docker.log
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

You'll also want to add the vars picked up to make all the files:

```
swatch_inits:
   - syslog
   - auth.log
   - docker.log
swatch_confs:
   - syslog
   - auth.log
   - docker.log
```

Example Playbook
----------------
```
---

- name: swatch
  hosts: all
  user: devopsdude
  remote_user: devopsdude
  sudo: yes
  roles:
    - ansible-swatch-role
  vars_files:
    - group_vars/swatch
```
Then run it something like:

`ansible-playbook -i inventory/other -u devopsdude -K ./swatch.yml`

Author Information
------------------

- LYRASIS
