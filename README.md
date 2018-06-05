# Tau Hypercomputing Facility
## Kerberos Key Distribution Center (KDC) service container

[![Release status: Pre-production][release-status]](#release-status)
![Classification: UNRESTRICTED][classification]
[![Apache 2.0-licensed][license]](#copyright)
[![Current Travis CI build status][travis]](https://travis-ci.org/tauproject/kdc)

This project builds a [Docker container image](https://hub.docker.com/r/tauproject/kdc/) for operating a [Heimdal](https://www.h5l.org) Kerberos Key Distribution Center (KDC).

### Release status

| Release track           | Tag/branch    |
|-------------------------|---------------|
| Long-term support (LTS) | Not available |
| Stable                  | Not available |
| Pre-production          | [`shimmer`](https://github.com/tauproject/kdc/tree/shimmer/master) |
| Legacy                  | Not available |

### Usage

It is assumed that you are familiar with operating a Kerberos KDC, and understand the security implications.

In the examples below, the `kadmind` service is explicitly bound to the IPv4 `localhost` address, 127.0.0.1 (i.e., `docker run ... -p 127.0.0.1:749:749/tcp ...`). In this configuration, other hosts will not be able to connect to the `kadmind` daemon running within the container.

On the container host, you can use `kadmin -a 127.0.0.1 ...` to force the `kadmin` utility to connect to that address without performing a realm admin server look-up. Alternatively, you can publish the port more widely if that is appropriate.

#### Pull the appropriate release image

Refer to the [*Release status*](#release-status) table above for information on available releases.

These examples will use the pre-production release tag, `shimmer`:

```
$ docker pull tauproject/kdc:shimmer
```

#### Starting the KDC for the first time

It is recommended the you start the KDC manually and interactively the first time you launch it, to confirm that the parameters you supply have the desired effect. The `docker run -it` option can be used for this:



```
# Note: substitute '/opt/kerberos/db' for your preferred location on the host
# for Kerberos database files.
#
# To avoid potential conflicts, it is strongly recommended that you do NOT use
# a location commonly used by KDC packages, such as '/var/lib/heimdal-kdc'.

$ mkdir -p /opt/kerberos/db
$ docker run -it \
    -e REALM=MYREALM -e HOSTNAME=$(hostname) \
    -p 88:88/tcp -p 88:88/udp \
    -p 464:464/udp \
    -p 127.0.0.1:749:749/tcp \
    -v /opt/kerberos/db:/app/db \
    tauproject/kdc:shimmer

[...output trimmed...]

kdc[MYREALM]: KDC started

[...KDC daemon output trimmed...]
```

In another terminal, you can test the KDC configuration. In the example invocation above, the standard KDC and password server ports are published to via the host’s interfaces.

Once you are satisfied that the KDC is running properly, you can shut it down with `docker stop`, or by pressing *Ctrl+C* in the terminal you started it from.

####  Normal KDC operation

To run the KDC normally, start it as you would any other Docker container. This example uses `docker run`, but you may wish to use a more complex orchestration mechanism on your hosts, such as [Puppet](https://forge.puppet.com/puppetlabs/docker).

```
$ docker run \
    -e REALM=MYREALM -e HOSTNAME=$(hostname) \
    -p 88:88/tcp -p 88:88/udp \
    -p 464:464/udp \
    -p 127.0.0.1:749:749/tcp \
    -v /opt/kerberos/db:/app/db \
    tauproject/kdc:shimmer
```

If this is the first time the container has been run with this realm name and database directory, a new realm database will be created in `/opt/kerberos/db/MYREALM` (in this example).

As part of this realm database initialiation, an `admin/admin@MYREALM` principal will be created and granted access to use `kadmin`; the password for this principal is written to `/opt/kerberos/db/MYREALM/admin-pw` and the `kadmin` ACL is written to `/opt/kerberos/db/kadmind.acl`.

Change the `admin/admin` password (or delete the principal altogether) and remove the `admin-pw` file from the realm database directory once the realm has been created.

#### Local `kadmin` mode

It is occasionally necessary to access a Kerberos database using `kadmin` in local mode, manipulating the database directly whilst the KDC and admin server daemons are not running.

The container supports this mode of operation by being launched with the `kadmin` command:

```
$ docker run -it \
    -e REALM=MYREALM  -e HOSTNAME=$(hostname) \
    -v /opt/kerberos/db:/app/db \
    tauproject/kdc:shimmer kadmin

[...output trimmed...]

kadmin>
```

The container will shut down once you exit `kadmin`.

#### Invoking a shell

You can launch the container and invoke a shell instead of running the KDC daemons by specifying the command `shell`:

```
$ docker run -it \
    -e REALM=MYREALM  -e HOSTNAME=$(hostname) \
    -v /opt/kerberos/db:/app/db \
    tauproject/kdc:shimmer shell
    
[...output trimmed...]

root@662e4c889f01:~#
```

#### Configuring replication

This container can be used as a source for [incremental propagation](https://www.h5l.org/manual/HEAD/info/heimdal/Incremental-propagation.html) to replicas.

1. Create `iprop/<hostname>@MYREALM`principals for each replica, and add their full principal names, one per line, to `/opt/kerberos/db/MYREALM/slaves`.
2. The `iprop/<hostname>@MYREALM` principal for the KDC is created automatically when the database is initialised and stashed in the container’s keytab each time it is launched. If you have migrated from another KDC, you may need to create this principal yourself.
3. Start the container as usual, but publish TCP port 2121 in addition to the usual Kerberos ports (for example, with `docker run -p 2121:2121 ...`)

#### Migrating existing data

**Note:** this procedure is experimental and insufficiently tested; **you proceed at your own risk**.

If you already have a Heimdal KDC, you can migrate to this container by taking the following steps:

1. Launch this container with a **new** database directory, specifying the name of your realm as described above, without publishing ports.
2. Once the container has launched successfully and the database files created, shut it down.
3. Shut down your *live* Heimdal KDC that you wish to migrate from.
4. Replace the files `heimdal.db`, `m-key`, `kadmind.acl` (and `slaves`, if employing incremental replication) in the database directory with those from your original KDC’s database directory. On Debian-based systems, these files will be found in  `/var/lib/heimdal-kdc`.
5. Optionally, edit `kdc.conf` in the database directory. Take care not to change the paths or logging options, or you may inhibit the proper operation of the container.
6. Start this container, publishing the standard Kerberos ports.
7. Confirm that you can authenticate correctly (e.g., by creating a custom `krb5.conf` on a client host directing KDC traffic only to this new container) and use `kadmin` as normal.
8. If your container's now running on a different host to your original KDC, update DNS or your clients' configurations to begin using your running container as a KDC.

**Note:** migrating data from a non-Heimdal KDC will require additional steps. Consult the [Heimdal documentation](https://www.h5l.org/manual/HEAD/info/heimdal/Migration.html) for further information. 

### Raising issues

If you discover a bug in this project, or have a request for a feature, please create a new [Github issue](https://github.com/tauproject/kdc/issues) describing it with as much detail as you're able to provide.

### Development

This section is intended for developers and administrators who may wish to contribute to the project or produce custom variants.

#### Source control

The public repository for this project is hosted on [Github](https://github.com/tauproject/kdc/). It may be cloned with:

```
$ git clone -b shimmer/master git://github.com/tauproject/kdc.git
```

or, if you prefer HTTPS:

```
$ git clone -b shimmer/master https://github.com/tauproject/kdc.git
```

#### Local builds

[GNU Autotools](http://www.gnu.org/software/automake/manual/html_node/Autotools-Introduction.html) are used to provide the build mechanics. Basic usage is described below.

##### Prerequisites

You must have a working Docker installation, as well as GNU Autoconf, GNU Automake, and a working POSIX-compatible `make` utility (such as GNU or BSD `make`).

Builds are routinely performed on:

* Debian GNU/Linux and Ubuntu hosts using tools installed via `apt`
* macOS hosts, using [Docker for Mac](https://www.docker.com/docker-mac), [the Apple Command Line tools](http://osxdaily.com/2014/02/12/install-command-line-tools-mac-os-x/), and [Homebrew](https://brew.sh)

##### Prepare the cloned repository

Once you have cloned the repository, you must use the `autoreconf` utility to prepare it for building:

```
$ cd kdc
$ autoreconf -fvi
autoreconf: Entering directory `.'
autoreconf: configure.ac: not using Gettext
autoreconf: running: aclocal --force -I m4
autoreconf: configure.ac: tracing
autoreconf: configure.ac: not using Libtool
autoreconf: running: autoconf --force
autoreconf: configure.ac: not using Autoheader
autoreconf: running: automake --add-missing --copy --force-missing
configure.ac:29: installing 'scripts/install-sh'
configure.ac:29: installing 'scripts/missing'
autoreconf: Leaving directory `.'
```

Note that the actual output from `autoreconf` may differ. You can omit the `-v` option to suppress the verbose output.

You should re-run `autoreconf` (and follow the configuration steps below) whenever you `git pull`, or if the `configure.ac` or any `Makefile.am` files are modified.

##### Configure the repository

Invoke the `configure` script:

```
$ ./configure
checking for a BSD-compatible install... /usr/bin/install -c
checking whether build environment is sane... yes
checking for a thread-safe mkdir -p... scripts/install-sh -c -d
checking for gawk... no

[...output trimmed...]

config.status: creating scripts/ci/Makefile
config.status: creating scripts/docker/Makefile
```

##### Performing builds

Run `make` at the top level of the repository to build a new container image:

```
$ make
```

##### Automated tests

Use `make check` to invoke any automated tests:

```
$ make check
```

These tests are also performed by the Travis CI jobs used to generate the release images.

##### Manual tests

The following targets are provided for testing; see [`docker/kdc/Makefile.am`](https://github.com/tauproject/kdc/blob/shimmer/master/docker/kdc/Makefile.am) for details of the parameters passed to `docker run`:

```
# Launch the container
$ make run

# Launch the container, running a shell instead of the KDC daemon
$ make shell

# Launch the container, running `kadmin -l` instead of the KDC daemon
$ make kadmin
```

In each case, a directory named `data` at the root of your repository clone will be used as the database directory by the container. The first time you use any of the above commands, it will be created if it does not exist and an `EXAMPLE.COM` realm database initialised within it.

#### Automated builds

##### Travis CI

[Travis CI](https://travis-ci.org/tauproject/kdc) performs a complete build using `docker` at least once per day on each release branch and is configured to push the resulting images to [Docker Hub](https://hub.docker.com/r/tauproject/kdc/). These images constitute the official releases.

##### Docker Cloud

Docker Cloud is also configured to perform an automated build. This is to ensure that the information presented on [Docker Hub](https://hub.docker.com/r/tauproject/kdc/) is up to date. The resulting `cloudbuild` tag should be ignored.

### Copyright

Copyright © 2018.

Licensed under the [Apache License, Version 2.0](http://www.apache.org/licenses/LICENSE-2.0) (the "License"); you may not use
this file except in compliance with the License.

You may obtain a copy of the License at:

http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software distributed
under the License is distributed on an **"as-is" basis, without warranties or
conditions of any kind**, either express or implied.  See the License for the
specific language governing permissions and limitations under the License.

```
@(#) $Tau: kdc/README.md $
```

[travis]: https://img.shields.io/travis/tauproject/kdc.svg?style=flat-square
[license]: https://img.shields.io/badge/license-Apache%202.0-blue.svg?style=flat-square
[release-status]: https://img.shields.io/badge/release%20status-Pre--production-yellow.svg?style=flat-square
[classification]: https://img.shields.io/badge/classification-UNRESTRICTED-brightgreen.svg?style=flat-square
