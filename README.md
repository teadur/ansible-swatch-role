Ansible Swatch Role
=========

An Ansible role to install Swatch: The Simple Log Watcher with a handy Slack/Irker integration & monit files.

Requirements
------------

Tested on Ubuntu Trusty and Precise. Uses apt to install things so it won't work on RedHat etc as it is written.

Role Variables
--------------

This role is designed to watch log files. Which files get watched gets defined by the `swatch_files` in your vars.

You'll also need to set {{ swatch_email_to }} somewhere to have email delivered.

This role will also post to Slack or Irker from swatch using curl in a shell script.

This gets around the need for creating a .conf and .init for every log file. Any log file can be added to swatch_files -- and defined per inventory.

e.g. For Slack:

Those vars can be used to build a shell script like this:

```
#!/bin/bash
ERROR=$1
curl -X POST --data-urlencode 'payload={"channel": "swatch", "username": "'$HOSTNAME'", "text": " Swatch error from : _'$HOSTNAME'_ : `'"'$ERROR'"'` ", "icon_emoji": "':$ICON:'"}' \
 {{ swatch_slack_webhook_url }}

```

That shell script ends up in `/usr/local/bin/slack-swatch.sh`

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

License
---

The project is available as open source under the terms of the [MIT License](http://opensource.org/licenses/MIT).

---
