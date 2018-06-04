# Tau Hypercomputing Facility
## Kerberos Key Distribution Center (KDC) service container

[![Current build status][travis]](https://travis-ci.org/tauproject/kdc)
[![Apache 2.0 licensed][license]](#copyright)

This project builds a container image for operating a [Heimdal](https://www.h5l.org) Kerberos Key Distribution Center (KDC).

### Usage

It is assumed that you are generally familiar with operating a Kerberos KDC.

Note that in the examples below, the `kadmin` service is explicitly bound to (IPv4) `localhost` (127.0.0.1). On the container host, you can use `kadmin -a 127.0.0.1 ...` to force a `kadmin` connection to `localhost`.

####  Normal KDC operation

```
$ mkdir -p /opt/kerberos/db
$ docker run \
    -e REALM=MYREALM \
    -p 88:88/tcp -p 88:88/udp \
    -p 464:464/udp \
    -p 127.0.0.1:749:749/tcp \
    -v /opt/kerberos/db:/app/db \
    tauproject/kdc:shimmer
```

If this is the first time the container has been run with a realm database will be created in `/opt/kerberos/db/MYREALM` (in this example). An `admin/admin@MYREALM` principal will be created and granted access to use  `kadmin`; the password for this principal is written to `/opt/kerberos/db/MYREALM/admin-pw`.

#### Local `kadmin` mode

```
$ docker run -it \
    -e REALM=MYREALM \
    -v /opt/kerberos/db:/app/db \
    tauproject/kdc:shimmer kadmin
```

The container will shut down once you exit `kadmin`.

### Caveats & limitations

1. Change the `admin/admin` password (or delete the principal altogether) and remove the `admin-pw` file from the realm database directory  once the realm has been created.
2. There is no provision for importing data from an existing KDC database.
3. Replication configuration is not currently orchestrated.

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

[travis]: https://img.shields.io/travis/tauproject/kdc.svg
[license]: https://img.shields.io/badge/license-Apache%202.0-blue.svg
