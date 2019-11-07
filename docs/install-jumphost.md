# Basic VoIP Service Setup

## Setup virtual data centre and VIMs

If you do not have an Openstack instance, a virtual sandbox can be setup on a bare metal machine by deploying an [NFVI underlay project](https://github.com/accanto-systems/nfvi-environment) or by installing a standard [DevStack environment](https://docs.openstack.org/devstack/latest/). 

## Setup LM

If you are using LM AIO project to install LM, then follow the following [guide](/docs/install-AIO.md).

## Configure locations in LM

Once AIO is up and running, log in and create the following locations. 

### Add openstack Core location

Add a location called "core" with resource manager "defaultRM" and infrastructure type "Openstack" and provide the following properties

```
os_auth_url: "http://192.168.10.10/identity/v3"
os_projectname: demo
os_username: demo
os_password: secret
almip: 192.168.10.5
```

Change the above with your Openstack credentials and your LM IP address. 

## Prepare Openstack

Create a key called "default".

The Jumphost and other images below depend on Openstack having a Xenial ubuntu image pre-loaded. Xenial cloud image can be downloaded from [here](https://cloud-images.ubuntu.com/xenial/current/xenial-server-cloudimg-amd64-disk1.img).

## Build Images and load VNFs/Network Services

The following Openstack images need to be built to support this scenario. 
* [IPPBX Openstack](/vnfs/ip-pbx/VNFCs/asterisk-vnfc/VDUs/packer/openstack/Readme.md)
* [Voip Gateway Openstack](/vnfs/voip-gateway/VNFCs/kamailio-vnfc/VDUs/packer/openstack/Readme.md)
* [SIPP Openstack](/vnfs/sip-performance/VNFCs/sipp-vnfc/VDUs/packer/openstack/Readme.md)

[Install LMCTL](/docs/install-lmctl.md) and load the VNFs and Network services into LM by running the following commands. 

```
cd marketplace
./load.sh push dev
```

You are now ready to run through the [basic demo scenario](/docs/basic-demo.md). 