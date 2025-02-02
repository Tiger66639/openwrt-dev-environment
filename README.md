# PirateBox OpenWRT development environment scripts
This is a collection of scripts and documentation for developing on scratch on PirateBox-OpenWRT packages, images and related things. All of the commands in the Makefile are customized or assume, that you don't rely on the current stable source.

## What this repository is
This repository is intended to get you started with a development environment to build your own PirateBox images - no matter if you just want to enable an additional OpenWRT feature, remaster your PirateBox image or start developing for the PirateBox.

## What this repository is not
* __NO__ newbie guide. You should know your way around in Makefiles you should also have some knowledge about building OpenWRT. In doubt follow the reference Links - if they do not help, feel free to file an issue.
* __SHOULD NOT__ be used if you only want to create a customized OpenWRT image. If you want to do that, use [openwrt-image-build](http://wiki.openwrt.org/doc/howto/obtain.firmware.generate) instead.

## Prerequisites
* Make sure you have the loop kernel module loaded:

        modprobe loop

* Make sure you have at least __8BG__ free disk space
* Make sure you have the following packages installed:
  * git
  * subversion
  * python3

## Setting up the development enviroment
There are two methods to build the image:
* Using the *official* piratebox feed
* Using a custom, local feed

Use the __local feed__ variant if you want to use __other__ branches __than__ the __master__ or __develepmont__ branch or if you want to pull in your own packages.

## For the impatient
The Makefile comes with __four auto build targets__, you start them, lean back and wait for the finished images. They at some point all require you to enter the root password, so you either need to wait until you are prompted to input your password or set your sudo timeout to unlimited, do some action as sudo in the current terminal and than start the build process.

* _make auto_build_stable_     
Will build the stable release with the **master branch** of the [openwrt-piratebox-feed](https://github.com/PirateBox-Dev/openwrt-piratebox-feed).

* _make auto_build_beta_     
Will build the beta release with the **development branch** of the [openwrt-piratebox-feed](https://github.com/PirateBox-Dev/openwrt-piratebox-feed).

* _make auto_build_development_    
Will build a snapshot release, using a local feed including the __develompment branches__ of packages otherwise pulled in via the [openwrt-piratebox-feed](https://github.com/PirateBox-Dev/openwrt-piratebox-feed).

* _make auto_build_local_     
Will build a release using all the packages from the _local_feed_ folder, but without changing brnaches. It will use all the packages at the set branch und build from there. This is the best way to implement and test your own changes, using exactly the branches you want.

## Detailed build instructions
Find below the steps described each of the automated targets uses.
The PirateBox feed variant is described in detail, other variants are described in how they differ from the PirateBox feed variant.

### PirateBox feed variant
To build your PirateBox image with the __master branch__ of the [openwrt-piratebox-feed](https://github.com/PirateBox-Dev/openwrt-piratebox-feed) execute the following steps in order:
    
1. Clone and configure OpenWRT and clone the image build script
    
        make openwrt_env
Detailed information about the OpenWRT build system may be found in the OpenWRT Wiki:

  * [build system](http://wiki.openwrt.org/doc/howto/buildroot.exigence)
  * [obtaining the source](http://wiki.openwrt.org/doc/howto/buildroot.exigence#downloading.sources)

2. Apply the PirateBox OpenWRT feed     
You can learn more about feeds on the [feeds page](http://wiki.openwrt.org/doc/devel/feeds) in the OpenWRT wiki.

        make apply_piratebox_feed
        
    or
        make apply_piratebox_beta_feed

3. Update all feeds

        make update_all_feeds

4. Install the PirateBox OpenWRT feed

        make install_piratebox_feed

5. Create the piratebox script image

        make create_piratebox_script_image

6. Build OpenWRT
If you have more than four cores, do not forget to adjust the __THREADS__ variable in the Makefile.     
This will copy the default kernel config and start building OpenWRT.

        make build_openwrt
The __THREADS__ variable in the Makefile needs to be adjusted to your system, a good rule of thumb for the value is to use the amount of cores you have available on your build machine.     
Building the OpenWRT image may take a long time, depending on your machine, up to a couple of hours.

7. Aquire missing packages    
There are a couple of packages that did not make it in the OpenWRT repo yet, so you need to acquire them manually:

        make acquire_stable_packages
        
    or
    
        make acquire_beta_packages

8. Start local repository    
After building OpenWRT you can start your local repository:

        make run_repository_all
This will start a python http server on port __2342__.
If you now surf to http://localhost:2342 you can verify that the repository is up and running.
If you want to change the port of the local repository, set it in the __Makefile__.

9. Build the PirateBox image     
To build the PirateBox image and istall.zip run:

        make piratebox

10. Stop the local repository
After building the image you can stop your local repository:

        make stop_repository_all

11. Enjoy your build     
You should now have a directory called __target_piratebox__ in the __openwrt-image-build__ directory.
This directory contains all supported firmware images and the install_piratebox.zip

You can now continue with the [auto installation](http://piratebox.cc/openwrt:diy) step.

### Local feed variant
The local feed variant only differs in a couple of steps from the piratebox feed method.
If you want to add your own repositories to the local feed, create a local feed directory

    mkdir local_feed

And add your repositories. Then run the steps from above.

Instead of __Step 2__ you run:

    make apply_local_feed

Instead of __Step 4__ you run:

    make install install_local_feed

Skip __Step 7__, because the packages downloaded in this step are build from source when using the local feed variant. 

### Cleaning up
There are two different clean targets:     

    make clean
    
Will clean the __openwrt-image-build__ directory and the __openwrt__ directory. It will however not delete the OpenWRT toolchain. It will also stop the local repository in case it is still running. This is also the first step executed when using any of the auto build targets.

    make distclean
    
Will do the same as clean, but will also delete all directories and files pulled in while building. The directory will basically look like after a fresh clone.

### Troubleshooting
#### Builing OpenWRT fails with errors
Run __make__ in the openwrt folder single threaded and with the __S=v__ flag to get detailed output:

    make S=v

### Benchmarks
This table is a short overview of build times on different systems. Initial is the time it took to build the first image, including toolchain. Following is the time it takes to build once the toolchain is available. Those values are by far not exact. Throughout the build process a lot of packages are fetched of the internet, so your speed also plays a big role. The times are intended to give you a bit of a feel of how long it will take you to build.

| CPU                         | Cores | Threads | RAM | Initial | Following |
|-----------------------------|:-----:|:-------:|:---:|--------:|----------:|
|Intel i7-4700MQ CPU @ 2.40GHz|8      |8        |8GB  |45:46    |28:22      |
