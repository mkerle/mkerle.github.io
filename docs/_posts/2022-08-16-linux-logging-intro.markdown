---
layout: single
title:  "Linux Logging Framework"
date:   2022-08-16 18:00:00 +1000
toc: true
categories: linux log syslog
---

# Introduction 

## Syslog Daemon

The Syslog daemon is reponsible for receiving local log information on a system.  

It can receive messages via the socket `/dev/log`.  The modern method to receive logs is via `systemd-journald`.

The daemon can write the log messages to file (usually in `/var/log/`) or log remotely.

There is no common format to use when sending logs to the syslog daemon.

## Standardising Logging Formats

There are a few RFC standards attempting to standardise log messages.
- RFC3164 - Community project
- RFC5424 - An official standard for log formats
- RFC5426 - Transport of log messages over TCP/UDP/514.  Logs can even be transported over HTTPS on TCP/6514.

`rsyslog` on Centos is capable of using standard log formats as per RFC3164 as found in `/etc/rsyslog.conf` (formats logs that go in `/var/log/messages`).

To do this a template is defined with the log format conforming to RFC3164.

## Linux Journal Logs

In modern Linux operating systems both `rsyslog` (or variant) and `systemd-journald` run side by side.  Typically rsyslog will read from journald and therefore there will be 2 copies of the log on the system.  Centos will use `imjournal` while Ubuntu will use `imuxsock`.

Journald logs can be viewed using `journalctl`.



# Log Processing

## Python Log Processing

An example python script for processing log files is shown here.  The script can look for standard log placement for both Centos and Ubuntu based systems.  The example script looks for ssh events.

{% highlight python %}
import re
import os.path

centos_ssh_log_file_path = "/var/log/secure"
ubuntu_shh_log_file_path = "/var/log/auth.log"

ssh_log_files = [centos_ssh_log_file_path, ubuntu_shh_log_file_path]
regex_valid_login  = 'sshd\[.*\]*Accepted password'	 
regex_failed_login = 'sshd\[.*\]*Failed password'

for log_file in ssh_log_files:
    if os.path.isfile(log_file) :
        with open(log_file, "r") as file:
            for line in file:
                for match in re.finditer(regex_valid_login, line, re.S):
                    print("[*] Valid SSH Login found:\n\t"   + line,end = '')
                    for match in re.finditer(regex_failed_login, line, re.S):
                        print("[!] INVALID SSH Login found:\n\t" + line,end = '')

{% endhighlight %}


A similar script could be done to process apache logs as per below:

{% highlight python %}
import re
import os.path

centos_apache_log_file_path = "/var/log/httpd/access_log"
ubuntu_apache_log_file_path = "/var/log/apache2/access.log"

apache_log_files = [centos_apache_log_file_path,ubuntu_apache_log_file_path]
regex = '([(\d\.)]+) - - \[(.*?)\] \"(.*?)\" (\d+) (\d+) \"(.*?)\" \"(.*?)\"'

for log_file in ssh_log_files:
    if os.path.isfile(log_file) :
        with open(log_file, "r") as file:
            for line in file:
                for match in re.finditer(regex_pattern, line, re.S):
                    log_line = (re.match(regex, line)).groups()
                    print(log_line)

{% endhighlight %}


## Processing Logs at Scale

To process logs using scripts in a large environment we could use tools such as Ansible.  Ansible uses YAML and an example shown below.  In this example we would create the script as created in the earlier section and point ansible to this path.  The configuration tells that the script should be executed under the root user.

{% highlight yaml %}
---
- name: logparser
  hosts: server01
  tasks:

   - name: list files in folder
     become: yes
     become_user: root
     script: /opt/tools/ssh_log_parser.py
     args:
        executable: python3
     register: output
   - debug: var=output.stdout_lines

{% endhighlight %}

The playbook could be executed by:

`$ ansible-playbook /path/to/playbook/log_parser.yml -u remoteuser --key-file='/home/user/.ssh/ansible_rsa' -K`