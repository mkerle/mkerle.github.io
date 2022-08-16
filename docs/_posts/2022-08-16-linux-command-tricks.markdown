---
layout: single
title:  "Linux Command Tricks"
date:   2022-08-16 19:00:00 +1000
toc: true
categories: linux commands
---

# Commands

## lsof

This command lists open files on a system.
- `-p` Use this flag to specify a process ID to check what files a process has open.  Check the man page to see the different options available for this flag.

## pgrep

This command can be used to get the process ID given some name or pattern (see the man page).

## journalctl

This command can be used to get log information from a linux system.
- `-u` flag allows a service to be specified to get logs for.
- `--since` Allows a timeframe to be specified to display logs from.
- `--until` Allows an end timestamp to be specified to display logs to.

`$ journalctl -u sshd.service --since "1 hour ago"`

# Combining Commands

# lsof and pgrep

We could use these commands to get a list of files open by some process.

`$ lsof -p $(pgrep syslogd)`



