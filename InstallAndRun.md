## Install Ceph 


Build and vstart.sh:

Prerequisites:

- OS with GCC 8.0 or above, aka Fedora

- or SCL (software collections library) enabled for Centos or RHEL

Building Ceph:

    git clone git@github.com:bu-528-sp19/Ceph-RGW-Prefetching.git ; cd ceph

Install all the dependencies to build Ceph from source.

    ./install_deps.sh

Tell cmake to only build the Mon, OSD, and RGW to cut down build times.

    ./do_cmake.sh -DWITH_CEPHFS=OFF -DWITH_DASHBOARD_FRONTEND=OFF -DWITH_RBD=OFF

All of the following commands will be run from the build directory.

    cd build

Make the vstart target that builds what is necessary to start up a ceph cluster with vstart.sh. Pass in -j(number of processors) to speed up your builds.

    make vstart -j`nproc`

vstart.sh:

To start a local ceph cluster for development purposes, the RGW Ceph developers use ceph/src/vstart.sh. The variables at the beginning of the command specify how many of each ceph daemon to bring up.

    MON=1 OSD=3 RGW=1 MGR=0 MDS=0 ../src/vstart.sh -n -d

There is a stop.sh script that stops the vstart cluster by killing all the ceph processes. The logs in the out directory can take up a lot of space and should be removed after a stop.sh as well.

    $ ../src/stop.sh

    $ rm out/*
``boto_request.py`` can be used to put/get/prefetch a file from Ceph.
To put a file in the Ceph
``python3 boto_request.py put FILE_NAME``
To get a file in the Ceph
``python3 boto_request.py get FILE_NAME``
To prefetch a file in the cache in Ceph
``python3 boto_request.py prefetch FILE_NAME``

