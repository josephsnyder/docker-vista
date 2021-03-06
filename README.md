# Dockerized VistA instances

## Requirements
This has only been tested using:

* Docker for Mac
* Docker on Linux (Thanks: George Lilly)
* Docker Toolkit on Windows 7 (Thanks: Sam Habiel)

## Pre-requisites
A working [Docker](https://www.docker.com/community-edition#/download) installation on the platform of choice.

## Note
You cannot use docker exec -it osehravista bash to get access to the mumps prompt. This is likely due to how docker permission schemes for shared memory access works. Instead always gain access via the ssh commands below

## Pre-built images
Pre-built images using this repository are available on [docker hub](https://hub.docker.com/r/krmassociates/)

### Running a Pre-built image
1) Pull the image

    ```
    docker pull krmassociates/osehravista # subsitute worldvista or vxvista if you want one of those instead
    ```

2) Run the image
  Non QEWD enabled images

    ```
    docker run -p 9430:9430 -p 8001:8001 -p 2222:22 -d -P --name=osehravista krmassociates/osehravista # subsitute worldvista or vxvista if you want one of those instead
    ```

QEWD enabled images: Add `-p 8080:8080` to access QEWD/Panorama at port 8080.

```
docker run -p 9430:9430 -p 8001:8001 -p 2222:22 -p 8080:8080 -p 9080:9080 -d -P --name=osehravista krmassociates/osehravista-qewd
```

## Docker Build

### Build Commands
1) Build the docker image

    ```
    docker build -t osehra .
    ```

2) Run the created image

    ```
    docker run -p 9430:9430 -p 8001:8001 -p 2222:22 -p 9080:9080 -d -P --name=osehravista osehra
    ```

## Build Steps for Caché installs
Caché will not have any pre-built images due to license restrictions and needing licensed versions of Caché to be used.

The Caché install assumes that you are using a pre-built CACHE.DAT and will perform no configuration to the CACHE.DAT. The default install is done with "minimal" security.

The initial docker container startup will take a bit of time as it needs to perform a workaround due to limitations of the OverlayFS that docker uses (see: https://docs.docker.com/engine/userguide/storagedriver/overlayfs-driver/#limitations-on-overlayfs-compatibility)

Also, many options (EWD, Panorama, etc) are not valid for Caché installs and will be ignored.

1) Copy the Caché installer (.tar.gz RHEL kit) to the root of this repository
2) Copy your cache.key to the cache-files directory of this repository
3) Copy your CACHE.DAT to the cache-files directory of this repository
4) Build the image

   ```
   docker build --build-arg flags="-c -b -s -p ./Common/pvPostInstall.sh" --build-arg instance="cachevista" --build-arg entry="/opt/cachesys" -t cachevista .
   ```

5) Run the image:

   ```
   docker run -p9430:9430 -p8001:8001 -p2222:22 -p57772:57772 -d --name=cache cachevista
   ```

### Build Options

#### instance
Default: `oshera`

The `instance` argument allows you to define the instance Name Space and directory inside the docker container.

Example:

    docker build --build-arg instance=vxvista -t vxvista .

#### flags
Default: `-c -b -s`

The `flags` argument allows you to adjust which flags you want to send to the autoInstaller.sh. This includes any post install script.

Example:

    docker build --build-arg flags="-y -b -s -a https://github.com/OSEHRA/vxVistA-M/archive/master.zip" -t vxvista .

Example:

    docker build --build-arg flags="-y -b -s -p ./Common/vxvistaPostInstall.sh" -t vxvista .

To see the flags that are available, execute the command 'autoInstaller.sh -h'

### entry
Default: `/home`

The `entry` argument allows you to adjust where docker looks for the entryfile.

Example:

    docker build --build-arg entry="/opt/cachesys" -t cache .

### Build Examples
Default: "OSEHRA VistA (YottaDB, no bootstrap, with QEWD and Panorama)"

    docker build -t osehra-vista .
    docker run -d -p 9430:9430 -p 8001:8001 -p 2222:22 -p 8080:8080 -p 9080:9080 --name=osehravista osehra-vista

Plan VI (Internationalized Version) OSEHRA VistA (YottaDB, UTF-8 enabled, no bootstrap, with QEWD and Panorama)

    docker build --build-arg flags="-byuma https://github.com/OSEHRA-Sandbox/VistA-M/archive/plan-vi.zip -p ./Common/ov6piko.sh" --build-arg instance="ov6" -t ov6 .
    docker run -d -p 2222:22 -p 8001:8001 -p 9430:9430 -p 8080:8080 -p 9080:9080 --name=ov6 ov6

WorldVistA (YottaDB, Panorama, no boostrap, skip testing):

    docker build --build-arg flags="-bymsa http://opensourcevista.net/NancysVistAServer/BetaWVEHR-3.0-Ver2-16Without-CPT-20181004/FileForDockerBuildWVEHR3.0WithoutCPT.zip -p Common/wvDemopi.sh" --build-arg instance="wv" -t wv .
    docker run -d -p 2222:22 -p 8001:8001 -p 9430:9430 -p 8080:8080 -p 9080:9080 --name=wv wv

vxVistA (YottaDB, no boostrap, skip testing, fix Kernel Routines and do post-install as well):

    docker build --build-arg flags="-ybsfa https://github.com/OSEHRA/vxVistA-M/archive/master.zip -p ./Common/vxvistaPostInstall.sh" --build-arg instance="vxvista" -t vxvista .
    docker run -d -p 2222:22 -p 8001:8001 -p 9430:9430 -p 9080:9080 --name=vxvista vxvista

VEHU (GTM, no bootstrap, skip testing, Panorama)

    docker build --build-arg flags="-g -f -b -s -m -a https://github.com/OSEHRA-Sandbox/VistA-VEHU-M/archive/master.zip" --build-arg instance="vehu" -t vehu .
    docker run -d -p 2222:22 -p 8001:8001 -p 9430:9430 -p 8080:8080 -p 9080:9080 --name=vehu vehu

VEHU Plan VI (Internationalized Version) (YottaDB, UTF-8 enabled, no bootstrap, skip testing, Panorama)

    docker build --build-arg flags="-bysuma https://github.com/OSEHRA-Sandbox/VistA-VEHU-M/archive/plan-vi.zip" --build-arg instance="vehu6" -t osehra/vehu6:201808 -t osehra/vehu6:latest .
    docker run -d -p 2222:22 -p 8001:8001 -p 9430:9430 -p 8080:8080 -p 9080:9080 --name=vehu osehra/vehu6

RPMS (RPMS, YottaDB, no boostrap, skip testing, and do post-install as well)

    docker build --build-arg flags="-w -f -y -b -s -a https://github.com/shabiel/FOIA-RPMS/archive/master.zip -p ./Common/rpmsPostInstall.sh" --build-arg instance="rpms" -t rpms .
    docker run -d -p 2222:22 -p 9100:9100 -p 9101:9101 -p 9080:9080 --name=rpms rpms

Caché Install with local DAT file. You need to supply your own CACHE.DAT and CACHE.key and .tar.gz installer for RHEL.  These files need to be added to the cache-files directories.

    docker build --build-arg flags="-c -b -s -p ./Common/pvPostInstall.sh" --build-arg instance="cachevista" --build-arg entry="/opt/cachesys" -t cachevista .
    docker run -p 9430:9430 -p 8001:8001 -p2222:22 -p57772:57772 -p 9080:9080 -d -P --name=cache cachevista


Caché Install with local DAT file to stop after exporting the code from the Cache instance. You need to supply your own CACHE.DAT and .tar.gz installer for RHEL.
When available, the system will also install a "cache.key" file when the system is built.  If it is not present, the extraction will be performed serially.
These files need to be added to the cache-files directories.

    docker build --build-arg flags="-c -b -x -p ./Common/foiaPostInstall.sh" --build-arg instance="cachevista" --build-arg entry="/opt/cachesys" -t cachevista .
    docker run -p 9430:9430 -p 8001:8001 -p2222:22 -p57772:57772 -p 9080:9080 -d -P --name=cache cachevista

To capture the exported code from the container and remove the Docker objects, execute the following commands:

    docker cp cache:/opt/VistA-M /tmp/ # no need to put a cp -r
    docker stop cache
    docker rm cache
    docker rmi cachevista

A [volume](https://docs.docker.com/storage/volumes/) could also be mounted to the container.


### Tagging an image to upload to Docker Hub
First, you need to login to Docker Hub using the command `docker login`.

MAKE SURE THAT WHAT YOU PUSH IS OPEN SOURCE CODE. Docker Hub is a public
resource. There are enterprise versions of Docker Hub; we are NOT providing
intructions for that here.

Then you need to tag your image. First find the image ID using `docker images`. For example,

    REPOSITORY           TAG                 IMAGE ID            CREATED             SIZE
    wv                   latest              4d088aa275ff        13 hours ago        6.06GB
    vehu                 latest              ed6175534a1c        20 hours ago        4.29GB
    vxvista              latest              3866735fcfc0        21 hours ago        2.96GB
    cachevista           latest              071386cb0e80        46 hours ago        15.4GB
    centos               latest              75835a67d134        11 days ago         200MB
    osehra/osehravista   latest              8d58b9b985d7        5 weeks ago         4.63GB
    hello-world          latest              e38bc07ac18e        6 months ago        1.85kB

So, if we want to push `wv` up, then we need to use the image ID `4d088aa275ff`
using our username on dockerhub plus the name of the image. If your username on
Docker Hub is `boo`, then you need to tag your image as follows:

    docker tag 4d088aa275ff boo/wv

If you plan to have more than one version of your images, you can use `:`
after the tag name. Not using a `:` automatically applies the version `latest`.
For example,

    docker tag 4d088aa275ff boo/wv:v3

... to deploy v3.

The last step is pushing to Docker Hub. To push the `latest` version, just
do this:

    docker push boo/wv

To push a specific version, you need to put the `:`.

    docker push boo/wv:v3

The push will take a long time depending on how fast your upload speed is. VistA
images are around 4GB big when uploaded; 1GB big when downloaded (as they are
downloaded gzipped).

### Building ViViaN and DOX with Docker

Utilizing the "-v" argument flag, the system will attempt to execute the tasks which will
install a MUMPS environment, execute tasks to gather data, generate HTML pages, and finally
set up a web server on the container to display the data.  The scripts are designed to
take and process a M[UMPS] system that is supplied by the user in one of two formats.

#### Files needed for building


|     Platform      |                       Required Files                           |
| :---------------: | -------------------------------------------------------------- |
|   GT.M/YottaDB    | A .zip file which contains a folder named "VistA" which        |
|                   | contains the GT.M  instance to generate pages from.            |
|                   | It should contain folder named ``g`` of globals (.GLD and .DAT)|
|                   | and a directory named ``r`` which contains the ``.m`` files    |
|                   | for the routines.  It should be placed in the ``GTM`` directory|                                               |
| ----------------- | -------------------------------------------------------------- |
|     Caché         | The files used as part of the install will be used again       |
|                   | You need to supply your own CACHE.DAT and CACHE.key and .tar.gz|
|                   | installer for RHEL.  These files need to be added to the       |
|                   | cache-files directories.                                       |


The building of ViViaN is available to executed on all three of the platforms using the same
arguments as above: ``-c`` for Caché, ``-y`` for YottaDB, and ``-g`` for GT.M.  Each of these
options should be combined with the ``-v`` and ``-b`` options when the docker build command is
instantiated.

*At this point, the instance must be named "osehra".*

For a Caché instance, the command would look as follows:

    docker build --build-arg flags="-c -b -v -p ./Common/pvPostInstall.sh" --build-arg entry="/opt/cachesys" --build-arg instance="osehra" -t cacheviv .
    docker run -p 9430:9430 -p 8001:8001 -p 8080:8080 -p 2222:22 -p 57772:57772 -p 3080:80 -d -P --name=cache cacheviv

For a YottaDB instance, the command would look as follows:

    docker build --build-arg flags="-y -b -v -p ./Common/pvPostInstall.sh" --build-arg instance="osehra" -t yottaviv .
    docker run -p 9430:9430 -p 8001:8001 -p 8080:8080 -p 2222:22 -p 57772:57772 -p 3080:80 -d -P --name=cache yottaviv

Once the container is running, the ViViaN and DOX pages can be accessed via
a web browser at http://localhost:3080/vivian and http://localhost:3080/vivian/files/dox

### Post Installs that you can apply with -p flag

| Script                              | What it does? |
| ----------------------------------- | ------------- |
| `./Common/pvPostInstall.sh`         | Generic Cache Set-up  |
| `./Common/syntheaPostInstall.sh`    | Install Synthetic Patient Generator |
| `./Common/vxvistaPostInstall.sh`    | vxVistA GT.M/YDB specific set-up |
| `./Common/rpmsPostInstall.sh`       | RPMS GT.M/YDB specific set-up |
| `./Common/foiaPostInstall.sh`       | Fix FOIA Console Set-up |
| `./Common/ov6pi.sh`                 | Add Korean demo data for Plan VI images |
| `./Common/ovydbPostInstall.sh`      | DO NOT USE |
| `./Common/wvDemopi.sh`              | Create Demo Users for an instance (physician, pharmacist, and nurse) |


### Installing SQL Mapping

SQL Mapping of FileMan Files is in development at https://github.com/YottaDB/PIP. SQL Mapping is supported for YottaDB and GT.M. There are some special command line arguments that are required for proper running:

#### Installation Command Line flag

The -q command line flag is used to install all required files for the SQL Mapping and set up processes for auto start when the container is started.
Development directories are automatically installed by specifing "-q"

An example build command:

    docker build --build-arg flags="-y -b -e -m -q -s" --build-arg instance="osehra" -t osehrapip .

#### Docker run Command Line flags

SQL Mapping uses POSIX message passing to perform certain operations and the Linux defaults are too small for operation. The following command line arguments are used to increase the size of the POSIX message passing parameters to support the SQL Mapping tool.

    --sysctl kernel.msgmax=1048700
    --sysctl kernel.msgmnb=65536000

There is also an additional port that needs to be forwarded from the Host to the Guest:

    -p 61012:61012 -p 8081:8081

example docker run command:

    docker run -p 9430:9430 -p 8001:8001 -p 2223:22 -p 61012:61012 -p 8081:8081 -d -P --sysctl kernel.msgmax=1048700 --sysctl kernel.msgmnb=65536000 --name=osehra osehrapip

#### Mapping FileMan Files

Since PIP has specific environment setup that is outside the normal from VistA it isn't integrated into the "normal" environemnt and must be ran using a helper script that sets up several environment variables for C Call-Outs and configuration.

    pip/dm

To map individual files:

    OSEHRA>D MAPFM^KBBOSQL(FileNumber)

Replace FileNumber with a valid parent File Number like 200 (NEW PERSON) or 2 (PATIENT)

    OSEHRA>D MAPFM^KBBOSQL(200)

Mapping all FileMan files can be accomplished by running:

    OSEHRA>D MAPALL^KBBOSQL

Note: This will corrupt certain tables in DATA-QWIK. See PIP issue: https://github.com/YottaDB/PIP/issues/72

If you don't use the above script errors in the M routine will be reported such as

    OSEHRA>D MAPFM^KBBOSQL(200)
    %GTM-E-UNDEF, Undefined local variable: QUIT
    At M source location CREATEMAP+19^KBBOSQL

or

    OSEHRA>D MAPFM^KBBOSQL(200)
    %GTM-E-UNDEF, Undefined local variable: PARENT
    At M source location GETPARENTS+1^KBBOSQL

## Roll-and-Scroll Access for non Caché installs

1) Tied VistA user:

    ssh osehratied@localhost -p 2222 # subsitute worldvistatied or vxvistatied if you used one of those images

password tied

2) Programmer VistA user:

    ssh osehraprog@localhost -p 2222 # subsitute worldvistaprog or vxvistaprog if you used one of those images

password: prog

3) Root access:

    ssh root@localhost -p 2222

password: docker

## VistA Access/Verify codes for non Caché installs

OSEHRA VistA:

Regular doctor:
Access Code: FakeDoc1
Verify Code: 1Doc!@#$

System Manager:
Access Code: SM1234
Verify Code: SM1234!!!

WorldVistA:

Displayed in the VistA greeting message

vxVistA:

Displayed in the VistA greeting message

## QEWD passwords for non Caché installs

Monitor:
keepThisSecret!

## Tests
Deployment tests are written using [bats](https://github.com/sstephenson/bats)
The tests make sure that deployment directories, scripts, RPC Broker, VistALink
are all working and how they should be.

There are two special tests:
 * fifo

   The fifo test is for docker containers and assumes that the tests are ran as root
   (currently) as that is who owns the fifo
 * VistALink

   This test installs java, retrieves a zip file of a github repo and makes a VistALink
   connection. This test does take a few seconds to complete and modifies the installed
   packages of the system. It also needs to have 2 environemnt variables defined: accessCode
   and verifyCode. These should be a valid access/verify code of a system manager user
   that has access to VistALink

