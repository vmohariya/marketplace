# Multiple VIMs with Provider Networking

This demonstration scenario deploys the VoIP network service to multiple VIM locations and types. The gateway and SIP simulator VNFs can be deployed both to Openstack or Kubernetes environments. Both VIM locations are connected over a common set of Provider networks. VNF Virtual Machines and/or Containers are attached to the same VLAN segments

![VoIP Service](/docs/images/provider-voip.PNG)

The Provider networking for the scenario logical setup below

![Provider VIM Overview](/docs/images/provider-multi-vim-overview.PNG)

Provider networks
* **Mgmt**: Cross VIM management network. vlan 5
* **Internal Voice**: Internal provide network to route external SIP/RTP traffic. vlan 50
* **External Voice**: External voice network. vlan 10

The VNF packages in this project use the following VIM networking features to connect VMs/Containers to the provider networks:
* Provider Neutron Networks
* Multus CNI Plugin

## Setup virtual data centre and VIMs

You can use an existing Openstack region, fabric and kubernetes cluster, or create your own virtual Openstack, Kubernetes and Networking fabric on a bare metal machine by deploying an [NFVI underlay project](https://github.com/accanto-systems/nfvi-environment). 

## Setup LM

For this scenario, LM must be attached to the provider networks to it can route traffic to/from the mgmt provider vlan. 

If you are using LM AIO project to install LM, then follow the following [guide](/docs/install-AIO.md). Otherwise skip this step and follow your LM instructions. 

## Configure locations in LM

Once AIO is up and running, log onto LM and create the following locations. 

### Add openstack Core location

Add a location called "core-provider" with resource manager "defaultRM" and infrastructure type "Openstack" and provide the following properties

```
os_auth_url: "http://192.168.10.10:5000/v3"
os_projectname: admin
os_username: admin
os_password: password
almip: 10.0.30.5
```

Change the above with your Openstack credentials and your LM Provider IP address. 

### Add k8s edge location

Add a location called "edge" with resource manager "defaultRM" with infrastructure type "Kubernetes" and provide the following properties

```
k8s_address: 192.168.10.50
k8s_ssh_user: vagrant
k8s_ssh_password: vagrant 
almip: 10.0.30.5
```

Change the above with your k8s cluster and LM Provider IP address. The IP address of LM Is the one reachable over the Provier network. 

## Prepare Openstack

Create a key called "default".

The Jumphost and other images below depend on Openstack having a Xenial ubuntu image pre-loaded. Xenial cloud image can be downloaded from [here](https://cloud-images.ubuntu.com/xenial/current/xenial-server-cloudimg-amd64-disk1.img).

## Load VNFs and Network Services

As required, build VNF images as per each VNF readme and load into target VIM.

The Jumphost and other images below depend on Openstack having a Xenial ubuntu image pre-loaded. Xenial cloud image can be downloaded from [here](https://cloud-images.ubuntu.com/xenial/current/xenial-server-cloudimg-amd64-disk1.img).

The following Openstack images need to be built to support this scenario. 
* [VoIP PBX Openstack](/vnfs/ip-pbx/VNFCs/asterisk-vnfc/VDUs/packer/openstack/Readme.md)
* [Voip Gateway Openstack](/vnfs/voip-gateway/VNFCs/kamailio-vnfc/VDUs/packer/openstack/Readme.md)
* [SIPP Openstack](/vnfs/sip-performance/VNFCs/sipp-vnfc/VDUs/packer/openstack/Readme.md)

## Prepare k8s

The following Docker are available on [Dockerhub](https://hub.docker.com/u/accanto) or can be built locally by following the links below: 
* [VoIP PBX Docker](/vnfs/ip-pbx/VNFCs/asterisk-vnfc/VDUs/packer/docker/Readme.md)
* [Voip Gateway Docker](/vnfs/voip-gateway/VNFCs/kamailio-vnfc/VDUs/packer/docker/Readme.md)
* [SIPP Docker](/vnfs/sip-performance/VNFCs/sipp-vnfc/VDUs/packer/docker/Readme.md)

## Accessing the k8s dashboard

To access the [kubernetes dashboard](https://github.com/kubernetes/dashboard/blob/master/docs/user/access-control/creating-sample-user.md), you need to first get and admin-user token by running the following command. 

```
kubectl -n kubernetes-dashboard describe secret $(kubectl -n kubernetes-dashboard get secret | grep admin-user | awk '{print $1}')
```

If you are using the kubeadm cluster with the NFVI project you will have to log into the kubeadm master virtual machine and configure kubectl with its credentials, as follows:

```
ssh vagrant@192.168.10.50
sudo su -
export KUBECONFIG=/etc/kubernetes/admin.conf
kubectl -n kubernetes-dashboard describe secret $(kubectl -n kubernetes-dashboard get secret | grep admin-user | awk '{print $1}')
```

Copy the generated token and paste into the dashboard UI. 

## Install LM packages

[Install LMCTL](/docs/install-lmctl.md) and load the VNFs and Network services into LM by running the following commands. 

```
cd marketplace
./load.sh push dev
```

You are now ready to run through the [provider demo scenario](/docs/provider-demo.md).
