ZnapZend 0.18.0
===============

[![Build Status](https://travis-ci.org/oetiker/znapzend.svg?branch=master)](https://travis-ci.org/oetiker/znapzend)
[![Coverage Status](https://img.shields.io/coveralls/oetiker/znapzend.svg)](https://coveralls.io/r/oetiker/znapzend?branch=master)
[![Gitter](https://badges.gitter.im/oetiker/znapzend.svg)](https://gitter.im/oetiker/znapzend)

ZnapZend is a ZFS centric backup tool. It relies on snapshot, send and
receive to do its work. It has the built-in ability to manage both local
snapshots as well as remote copies by thinning them out as time progresses.

The ZnapZend configuration is stored as properties in the ZFS filesystem
itself.

Note that while recursive configurations are well supported to set up
backup and retention policies for a whole dataset subtree under the dataset
to which you have applied explicit configuration, at this time pruning of
such trees ("I want every dataset under var except var/tmp") is not supported.
You probably do not want to enable ZnapZend against the root datasets of your
pools due to that, but would have to be more fine-grained in your setup.


Zetup Inztructionz
------------------

Follow these zimple inztructionz below to get a custom made copy of
znapzend. Yes, you need a compiler and stuff for this to work.

On RedHat you get the necessaries with:

    yum install perl-core

On Ubuntu / Debian with:

    apt-get install perl unzip

On Solaris you may need the C compiler from Solaris Studio and gnu-make
since the installed perl version is probably very old.

On OmniOS/SmartOS you will need perl and gnu-make.

With that in place you can now utter:

```sh
wget https://github.com/oetiker/znapzend/releases/download/v0.18.0/znapzend-0.18.0.tar.gz
tar zxvf znapzend-0.18.0.tar.gz
cd znapzend-0.18.0
./configure --prefix=/opt/znapzend-0.18.0
```

If configure finds anything noteworthy, it will tell you about it.  If any
perl modules are found to be missing, they get installed locally into the znapzend
installation. Your perl installation will not get modified!

```sh
make
make install
```

Optionally (but recommended) put symbolic links to the installed binaries in the
system PATH.

```sh
for x in /opt/znapzend-0.18.0/bin/*; do ln -s $x /usr/local/bin; done
```

Debian packages
---------------

Debian control files, guide on using them and experimental debian packages can be found at https://github.com/Gregy/znapzend-debian


Configuration
-------------

Use the [znapzendzetup](doc/znapzendzetup.pod) program to define your backup
settings. They will be stored directly in dataset properties, and will cover
both local snapshot schedule and any number of destinations to send snapshots
to (as well as potentially different retention policies on those destinations).
You can enable recursive configuration, so the settings would apply to all
datasets under the one you configured explicitly.

For remote backup, znapzend uses ssh. Make sure to configure password-free
login (authorized keys) for ssh to the backup target host with an account
sufficiently privileged to manage its ZFS datasets under a chosen destination
root.

For local or remote backup, znapzend can use mbuffer to level out the bursty
nature of ZFS send and ZFS receive features, so it is quite beneficial even
for local backups into another pool (e.g. on removable media or a NAS volume).
It is also configured among the options set by znapzendzetup per dataset.
Note that in order to use larger (multi-gigabyte) buffers you should point
your configuration to a 64-bit binary of the mbuffer program. Sizing the
buffer is a practical art, depending on the size and amount of your datasets
and the I/O speeds of the storage and networking involved. As a rule of thumb,
let it absorb at least a minute of I/O, so while one side of the ZFS dialog
is deeply thinking, another can do its work.

Running
-------

The [znapzend](doc/znapzend.pod) daemon is responsible for doing the actual backups.

To see if your configuration is any good, run znapzend in noaction mode first.

```sh
znapzend --noaction --debug
```

If you don't want to wait for the scheduler to actually schedule work, you can also force immediate action by calling

```sh
znapzend --noaction --debug --runonce=<src_dataset>
```

then when you are happy with what you got, start it in daemon mode.

```sh
znapzend --daemonize
```

Best practice is to integrate znapzend into your system startup sequence, but you can also
run it by hand. See the [init/README.md](init/README.md) for some inspiration.

Troubleshooting
---------------

By default a znapzend daemon would log its progress and any problems to
local syslog as a daemon facility, so if the service misbehaves - that is
the first place to look. Alternately, you can set up the service manifest
to start the daemon with other logging configuration (e.g. to a file or
to stderr) and perhaps with debug level enabled.

Statistics
----------

If you want to know how much space your backups are using, try the
[znapzendztatz](doc/znapzendztatz.pod) utility.

Support and Contributions
-------------------------
If you find a problem with znapzend, please open an Issue on GitHub.

If you'd like to get in touch, come to [![Gitter](https://badges.gitter.im/oetiker/znapzend.svg)](https://gitter.im/oetiker/znapzend).

And if you have a contribution, please send a pull request.

Enjoy!

Dominik Hassler & Tobi Oetiker
2018-04-13
