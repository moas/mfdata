## What is it?

This is the **M**etwork **F**ramework **DATA** processing **module**. This module is a "production ready" framework
for processing "incoming files".

*Incoming files* can arrive by:

- system FTP
- system SCP
- Restful HTTP
- MQTT
- AMQP
- [...] (you can provide a "listener plugin" to support other things)

These "incoming files" are transient. The goal in to do something with them
**in the stream**.

To do that, you can easily code and provide **plugins** to build a
workflow. This workflow can be really easy and stupid (for example: do something
for every incoming files) but also really complex with hundreds of plugins, routing rules,
plugin sequences, file duplications...

To help with complex workflows, you can rely on :

- "loose coupling" and "dynamic business routing rules" between plugins thanks to the provided **switch plugin**
- "context transmission" thanks to the **xattrfile** library
- "full inspection support" in case of errors thanks to **failure policy**, **xattrfile** libraries and **archive plugins**
- "retry mechanisms" thanks to **failure policy** and **reinject plugins**
- "full monitoring" thanks to embedded metrics and logs management (and **mfadmin module**)

It's designed to process terrabytes of data every day in a very robust way:

- you can choose the parallelism level of each plugin with a configuration file
- plugins are automatically restarted if necessary
- "waiting queues" and "resources limits" protect the system for overloading

Last but not least, thanks to the "no polling at all" principle, you can process
several thousands of file by second.

## What is not in?

Of course, you can code the behavior you want but **mfdata module** is
mainly designed to be "stateless". If you want to store durably something,
have a look at **mfbase module**.

## Quickstart

### Installation

**On a Linux CentOS 7 box** (please refer to reference documentation for other Linux distributions)

```console
# AS ROOT USER

# First, we configure the Metwork Framework repository for stable release on CentOS 7
cat >/etc/yum.repos.d/metwork.repo <<EOF
[metwork_stable]
name=MetWork Stable
baseurl=http://metwork-framework.org/pub/metwork/releases/rpms/stable/centos7/
gpgcheck=0
enabled=1
metadata_expire=0
EOF

# Then we install a minimal version of mfdata module
yum -y install metwork-mfdata-minimal

# Done :-)

# Then let's install (for the example only) an extra layer
# (to add Python2 support)
yum -y install metwork-mfdata-layer-python2
```

If you need some additional libraries,
please have a look at [mfext](https://github.com/metwork-framework/mfext/README.md)
to learn how to find and install extra mfext layers.

Then, to start mfdata services:

```console
# AS ROOT USER

service metwork start mfdata
```

### Usage

```console
$ # AS mfdata user (created automatically during installation)
$ # (you can also work with a standard user, please refer to reference documentation
$ #  if you want to do that)

           __  __      ___          __        _
          |  \/  |    | \ \        / /       | |
          | \  / | ___| |\ \  /\  / /__  _ __| | __
          | |\/| |/ _ \ __\ \/  \/ / _ \| '__| |/ /
          | |  | |  __/ |_ \  /\  / (_) | |  |   <
          |_|  |_|\___|\__| \/  \/ \___/|_|  |_|\_\

Welcome on kalmar (kalmar.meteo.fr, 137.129.5.170)
(module: MFDATA, version: integration.ci427.a9e916f)

 17:08:24 up 348 days,  4:29,  1 user,  load average: 0.08, 0.02, 0.01

$ # Let's bootstrap an "archive all" plugin (from a template)
$ bootstrap_plugin.py create --no-input --template=archive archive_all
Plugin archive_all successfully created on directory /home/mfdata/archive_all

$ # Install it in dev mode
$ cd archive_all
$ make develop
[...]

$ # See installed plugins
$ plugins.list
┌Installed plugins (2)────────────────────┬──────────┬──────────────────────────────────────────────────┐
│ Name        │ Version                   │ Release  │ Home                                             │
├─────────────┼───────────────────────────┼──────────┼──────────────────────────────────────────────────┤
│ switch      │ integration.ci427.a9e916f │ 1        │ /home/mfdata/var/plugins/switch                  │
│ archive_all │ dev_link                  │ dev_link │ /home/mfdata/var/plugins/archive_all             │
└─────────────┴───────────────────────────┴──────────┴──────────────────────────────────────────────────┘

$ # Inject a file in default "incoming directory"
$ date > ~/var/in/incoming/foobar
$ ls ~/var/in/incoming/
$ # Empty !

$ # Examine the archive
$ # ls ~/var/archive/
20190924
$ cd ~/var/archive/20190924
$ ls
16fbc7a3bac94aa18e6fef62cc8ad474  16fbc7a3bac94aa18e6fef62cc8ad474.tags
$ cat 16fbc7a3bac94aa18e6fef62cc8ad474 # our file
Tue Sep 24 17:19:28 CEST 2019
$ cat 16fbc7a3bac94aa18e6fef62cc8ad474.tags # the full traceability in the mfdata worlflow
0.switch.main.enter_step = 2019-09-24T15:19:28:649642
0.switch.main.exit_step = 2019-09-24T15:19:28:651775
0.switch.main.process_status = ok
0.switch.main.system_magic = ASCII text
1.archive_all.main.enter_step = 2019-09-24T15:19:28:655054
first.core.original_basename = foobar
first.core.original_dirname = incoming
first.core.original_uid = 2d3228e270d7433891ce7fbfb67b9520
latest.core.step_counter = 1
latest.switch.main.system_magic = ASCII text
```
