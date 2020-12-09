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

## Get action images

The last step for the Provisioner to do at this point is to pull down the images that will be used in the workflow.
Tinkerbell uses Docker registry to host images locally, so pull down the various docker image and push it to the registry:

```bash
docker login docker.pkg.github.com

for image in disk-wipe disk-partition install-root-fs cloud-init
do
    docker pull docker.pkg.github.com/qmfrederik/ubuntu-workflow/$image:410149153
    docker tag docker.pkg.github.com/qmfrederik/ubuntu-workflow/$image:410149153 192.168.1.1/ubuntu-workflow/$image:latest
    docker push 192.168.1.1/ubuntu-workflow/$image:latest
done
```

## Verify the Worker hardware profile

You've previously created a hardware profile for the Worker VM.
You can retrieve the UDID of that hardware profile using the `think hardware list` command:

```
docker exec -i deploy_tink-cli_1 tink hardware list
```

If you haven't created a hardware profile yet, create one now:

```
cat /vagrant/hardware-data.json | docker exec -i deploy_tink-cli_1 tink hardware push
```

## Create a template which wipes and partitions the disk

```
cat wipe-and-partition.tmpl | docker exec -i deploy_tink-cli_1 tink template create -n wipe_and_partition_disk
```

Then, create a workflow which applies the template:

```
docker exec -i deploy_tink-cli_1 tink workflow create -t 65e8ac1d-3997-11eb-9c7d-0242ac120004 -r '{"device_1":"08:00:27:00:00:01"}'
```