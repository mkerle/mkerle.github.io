---
layout: single
title:  "Linux Logging Framework"
date:   2022-08-16 18:00:00 +1000
toc: true
categories: linux log syslog
---

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
