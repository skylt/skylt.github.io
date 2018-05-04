---
layout: post
title:  "Expect to automatize hp-plugin"
categories: cups docker expect
---

So, after some time without printers, we decided to have a cups server running
again.
On the contrary of the previous version, we wanted it to run in a
container. This implies several changes from a VM.

Firstly, we want it to be re-runnable. The launch must therefore be automatic,
and deterministic. On the other hand, we have hp printers, which forces us to
use hp-plugin. We need to find an easy way to automatize it.

[expect](https://linux.die.net/man/1/expect) seems to be the appropriate call
for this task, for example:
    `expect  -c spawn hp-plugin -i; expect "?" {send "\r"; exp_continue}`

After a little more research, it appears that [autoexpect](https://linux.die.net/man/1/autoexpect)
can help us even more. After running it with the program you want to automatize,
it will create an expect script that contains your interaction with the given
program.

This gives us :
```

set force_conservative 0  ;# set to 1 to force conservative mode even if
                          ;# script wasn't run conservatively originally
if {$force_conservative} {
        set send_slow {1 .1}
        proc send {ignore arg} {
                sleep .1
                exp_send -s -- $arg
        }
}

set timeout -1
spawn hp-plugin -i
match_max 100000
expect -exact "\r
[01mHP Linux Imaging and Printing System (ver. 3.16.11)[0m\r
[01mPlugin Download and Install Utility ver. 2.1[0m\r
\r
Copyright (c) 2001-15 HP Development Company, LP\r
This software comes with ABSOLUTELY NO WARRANTY.\r
This is free software, and you are welcome to distribute it\r
under certain conditions. See COPYING file for more details.\r
\r
[35;01mwarning: It is not recommended to run 'hp-plugin' in a root mode.[0m\r
\r
[01mHP Linux Imaging and Printing System (ver. 3.16.11)[0m\r
[01mPlugin Download and Install Utility ver. 2.1[0m\r
\r
Copyright (c) 2001-15 HP Development Company, LP\r
This software comes with ABSOLUTELY NO WARRANTY.\r
This is free software, and you are welcome to distribute it\r
under certain conditions. See COPYING file for more details.\r
\r
[31;01merror: PolicyKit support requires DBUS or PolicyKit support files missing[0m\r
(Note: Defaults for each question are maked with a '*'. Press <enter> to accept the default.)\r
\r
\r
------------------------------------------\r
| PLUG-IN INSTALLATION FOR HPLIP 3.16.11 |\r
------------------------------------------\r
\r
The driver plugin for HPLIP 3.16.11 appears to already be installed.\r
[01mDo you wish to download and re-install the plug-in? (y=yes*, n=no, q=quit) ? [0m"
send -- "y\r"
expect -exact "y\r

  Option      Description                                       \r
  ----------  --------------------------------------------------\r
  d           Download plug-in from HP (recommended)            \r
  p           Specify a path to the plug-in (advanced)          \r
  q           Quit hp-plugin (skip installation)                \r
[01m\r
Enter option (d=download*, p=specify path, q=quit) ? [0m"
send -- "d\r"
expect -exact "d\r
\r
-------------------\r
| DOWNLOAD PLUGIN |\r
-------------------\r
\r
Checking for network connection...\r
\Downloading plug-in from: \r
Downloading plug-in: \[\\                                                   \] 0%    \\\\Receiving digital keys: /usr/bin/gpg \r
\r
[31;01merror: Unable to recieve key from keyserver[0m\r
[01mDo you still want to install the plug-in? (y=yes, n=no*, q=quit) ? [0m"
send -- "y\r"
expect -exact "y\r
\r
----------------------\r
| INSTALLING PLUG-IN |\r
----------------------\r
\r
Creating directory plugin_tmp\r
Verifying archive integrity... All good.\r
Uncompressing HPLIP 3.16.11 Plugin Self Extracting Archive................................................\r
\r
[01mHP Linux Imaging and Printing System (ver. 3.16.11)[0m\r
[01mPlugin Installer ver. 3.0[0m\r
\r
[... Licence ...]
[01m\r
Do you accept the license terms for the plug-in (y=yes*, n=no, q=quit) ? [0m"
send -- "y\r"
expect eof
```

We should note here that all the dots correspond to a progress bar.
Therefore it would be smart to remove them to be sure the script works
regardless of the time it takes hp-plugin to run.

For that, we removed all text up to the progress bars. Which gives us:

```
set timeout -1
spawn hp-plugin -i
match_max 100000
expect "?"
send -- "d\r"
expect "?"
send -- "y\r"
expect "?"
send -- "y\r"
expect eof
```

So much shorter than expected.

For licence purposes, only run this script if you have read the hp-plugin licence.
