# Kubernetes cluster
A vagrant script for setting up a Kubernetes cluster using Kubeadm

## Pre-requisites

 * **[Vagrant 2.1.4+](https://www.vagrantup.com)**
 * **[Virtualbox 5.2.18+](https://www.virtualbox.org)**

## How to Run kube cluster

Execute the following vagrant command to start a new Kubernetes cluster, this will start one master and two nodes:

```
vagrant up
```

You can also start invidual machines by vagrant up k8s-head, vagrant up k8s-node-1 and vagrant up k8s-node-2

If more than two nodes are required, you can edit the servers array in the Vagrantfile

```
servers = [
    {
        :name => "k8s-node-3",
        :type => "node",
        :box => "ubuntu/xenial64",
        :box_version => "20180831.0.0",
        :eth1 => "192.168.205.13",
        :mem => "2048",
        :cpu => "2"
    }
]
 ```

As you can see above, you can also configure IP address, memory and CPU in the servers array.

## Setting up spinnaker on the local cluster

Steps are outlined [HERE](https://www.spinnaker.io/setup/install/halyard/)

Commands:
  ```
  curl https://raw.githubusercontent.com/spinnaker/halyard/master/install/macos/InstallHalyard.sh -o /tmp/install_halyard.sh

  sudo bash /tmp/install_halyard.sh

  hal -v

  hal config provider kubernetes enable

  CONTEXT=$(kubectl config current-context)
  export ACCOUNT=my-k8s-v2-account

  hal config provider kubernetes account add $ACCOUNT \
    --provider-version v2 \
    --context $CONTEXT

  hal config features edit --artifacts true

  hal config deploy edit --type distributed --account-name $ACCOUNT

  #reduce rbac restrictions...dont do this in prod, but needed for helm installs
  kubectl create clusterrolebinding permissive-binding --clusterrole=cluster-admin --user=admin --user=kubelet --group=system:serviceaccounts;

  brew install minio

  #Start minio service
  minio server /tmp/data &

  #record startup info...apparently not in stdout or stderr...Endpoint, AccessKey, SecretKey

  #turn off s3 versioning in spinnaker

  mkdir -p ~/.hal/default/profiles/
  echo "spinnaker.s3.versioning: false" > ~/.hal/default/profiles/front50-local.yml

  echo $MINIO_SECRET_KEY | hal config storage s3 edit --endpoint $ENDPOINT \
    --access-key-id $MINIO_ACCESS_KEY \
    --secret-access-key # will be read on STDIN to avoid polluting your
                        # ~/.bash_history with a secret

  hal config storage edit --type s3

  hal version list

  hal config version edit --version $VERSION

  hal deploy apply

  hal deploy connect
  ```
##LEFT OFF AT CONFIGURING STORAGE
https://www.spinnaker.io/setup/install/storage/

## Cluster Clean-up

Execute the following command to remove the virtual machines created for the Kubernetes cluster.
```
vagrant destroy -f
```

You can destroy individual machines by vagrant destroy k8s-node-1 -f

## Licensing

[Apache License, Version 2.0](http://opensource.org/licenses/Apache-2.0).
