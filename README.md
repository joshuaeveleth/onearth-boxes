NASA Global Imagery Browse Services (GIBS)
=======

This software is part of NASA GIBS OnEarth (https://github.com/nasa-gibs/onearth) and was developed at the Jet Propulsion Laboratory.

# OnEarth-Boxes
OnEarth-Boxes is a system that creates VM images for use in running, testing, and experimenting with the OnEarth and MRF tools. It uses [Packer](http://packer.io) to produce a variety of pre-built VMs in different formats.

Boxes comes with some sample imagery and some pre-configured endpoints for use with testing and development.

# What's in the Boxes?
OnEarth-Boxes comes pre-configured with NASA Blue Marble and MODIS Aerosol imagery layers, in 4 separate projections. There are OpenLayers endpoints for testing with WMTS, as well as TWMS/KML and Mapserver endpoints.

All the source code for both MRF and OnEarth is included, and all the utilities are pre-installed, including the GDAL MRF extensions.

To get started with OnEarth-Boxes once you've created a VM image and have it running, go to `<onearth-boxes_vm_url>/onearth/demo`.

# Requirements
Creation of an OnEarth-Boxes image requires [Packer](http://packer.io) to be installed on your computer.

# Creating an OnEarth VM
To create a VM, use the `packer build <template.json>` command. There are `.json` template files for several different kinds of builds. See below for details.

## Build Options
To specify options for the build process, use the `-var` tag, for example:

```packer build -var "host_port=8888" -var "repo_url=https://github.com/nasa-gibs/onearth.git" -var "repo_branch=v0.8.0" oe-demo.json```

### Available Options

`host_port` - in order for the OnEarth TWMS endpoint to work, Apache in the VM needs to be configured to use the same port within the VM as will be used on the host machine. **Defaults to 8080.**

In other words, if you're planning to access the VM under `localhost:8888`, it works best if Apache within the VM also uses that port. This option automatically configures Apache to use the specified port.

`repo_url` - Use this option to specify the repo Packer will clone to build OnEarth. Default is **[https://github.com/nasa-gibs/onearth.git](https://github.com/nasa-gibs/onearth.git)**.

`repo_branch` - Use this option to specify the version of OnEarth you want to install. This tag will be checked out before the build starts. Default is the latest OnEarth release. **Default is latest release (currently v0.8.0).**

**Note that using older versions of OnEarth may require tweaks to the `bootstrap.sh` script!**

## Builders
The following build templates are included:

- `oe-virtualbox.json` -- A VirtualBox image
- `oe-vagrant.json` -- A VirtualBox image packaged as a Vagrant box 
- `oe-vmware.json` -- A VMWare image
- `oe-parallels.json` -- A Parallels image
- `oe-docker.json` -- A Docker container


### Note for building Docker containers
Currently, Packer only supports building Docker containers in Linux. You'll need to install [Docker](http://www.docker.com/) and [Packer](http://packer.io) and make sure the Docker daemon is active before running the Packer build process.

----
## Vagrant info
Using [Vagrant](https://www.vagrantup.com) is one of the easiest ways to get started with the OnEarth demo VM.

### Step 1: Install [Vagrant](https://www.vagrantup.com)
Vagrant is available for Mac, Windows, and Linux. It's free and requires [VirtualBox](https://www.virtualbox.org/).

### Step 2: Build the Vagrant box with Packer
Run the default Packer command: `packer build oe-vagrant.json` within the root of this repo. See above for options to customize the install.

**Note that the build process compiles a lot of software and generates some MRF imagery, so it can take quite a while.**

### Step 3: Add the Vagrant box
After the Packer build process is complete, go to the `builds` directory that will be created in the root of the repo and add the box with this command:
`vagrant box add --name=onearth-demo virtualbox-onearth.box`

**Once the Vagrant box is added, you can create multiple new virtual machines using that box as a base. It's not necessary to rebuild with Packer each time.**

### Step 4: Create a Vagrantfile
From any directory you like, type the command `vagrant init`, which will set up a sample `Vagrantfile`. Open the Vagrantfile and make sure the following lines are present:

```config.vm.box = "onearth-demo"```

```config.vm.network "forwarded_port", guest: <chosen_port>, host: <chosen_port>```

### Step 5: Start the VM
Use the `vagrant up` command to boot the VM. The demo should now be available at: `localhost:<chosen_port>/onearth/demo`

You can use the `vagrant ssh` command to open a shell inside the VM. The directory that contains your `Vagrantfile` is mapped to `/vagrant` within the VM by default.

**The username and password within the Vagrant VM are `vagrant` and `vagrant`**

----
## Docker Info
[Docker](docker.com) is a great way to run OnEarth as a standalone process

To run the tests, first create a directory for storage of the test results. This directory needs open read/write privileges.

Use the following syntax to create a Docker container and run the tests. The container will exit and stop when the tests are complete.
`sudo docker run -v <location_of_test_output>:/etc/onearth/config/test/test_results <onearth_image_name> /etc/onearth/config/test/run_tests.sh`

If the tests fail, the output will be stored in a file called `test_error_log` in the test output directory you specified in the `docker run` command. Otherwise, no output file will be created.
