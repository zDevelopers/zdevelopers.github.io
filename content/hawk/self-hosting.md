+++
title = "Self-hosting Hawk"
description = "Self-hosting Hawk — Your reports on your own domain"

[extra]
menu_title = "Self-hosting"
+++

The website part of Hawk, [called `Hawh-GUI`](https://github.com/zDevelopers/Hawk-GUI), is written in Python (with
Django), alongside a Rust module to parse and process reports.

To deploy the main Hawk instance, [we have an Ansible playbook](https://github.com/zDevelopers/HawkGUI-Ansible). You can
use it to deploy to your own server. The playbook is adapted to our own server (running CentOS 7 and systemd), but it
would be easy to adapt for another by updating the playbook inventory (in `group_vars`) and maybe updating system
packages on `tasks/system.yml`.

Check out the README of the Ansible repo above for details. Fell free to [get in touch](/support) if you need help.
