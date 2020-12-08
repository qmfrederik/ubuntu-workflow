# Deploy Ubuntu 20.04 to the worker VM in the Tinkerbell sandbox

## Get the Ubuntu scripts

Now that Tinkerbell is up and running, you can configure the provisioner to support
deploying Ubuntu to the worker machine.

SSH into the provisioner:

```bash
vagrant ssh provisioner
>
vagrant@provisioner:~$
```

Clone the ubuntu-workflow repository into your home directory:

```
cd ~
git clone https://github.com/qmfrederik/ubuntu-workflow
```

## Get apt-cacher-ng

The Ubuntu provisioning workflow uses [grml-debootstrap](https://grml.org/grml-debootstrap/)
to deploy Ubuntu to the target machine.

Ubuntu is installed by fetching the Ubuntu packages directly from the Ubuntu repositories,
rather than using an Ubuntu installer image.

Because this has the potential to consume a lot of bandwidth, we'll use [apt-cacher-ng](https://wiki.debian.org/AptCacherNg)
to cache the Ubuntu packages on the provisioner VM.

apt-cacher-ng is going to be running from a container, so navigate to the ubuntu-workflow directory and start the
apt-cacher-ng image with docker-compose.

```bash
cd ~/ubuntu-workflow
docker-compose up -d
```
