# OCP4 on VMware vSphere IPI Automation

OpenShift 4.5 brings with it support for vSphere IPI (Installer Provisioned Infrastructure). This new support extends the functionality of what OpenShift can do from a Cloud Provider standpoint including cluster scaling (both manual and automatically) from within OpenShift.

See the OpenShift 4.5 install documentation for the requirements and process for installing OpenShift in this manner. If you are installing OpenShift IPI in a restricted environment there are still some pre-steps that need to be done. In addtion if you need support for internal NTP servers, we have a playbook for you!

For restricted networks (those networks that can not directly reach the Internet to pull container images) the oc command supports the ability to mirror both the OpenShift install images as well as the Operator Hub and its required images.

## Requirements

* a host with ansible 2.9 installed
* host that can access the internet, as well as your target image repo and the subnet you will deploy to
* a subnet with DHCP _enabled_
* two Staticly assigned IP addresses on the same subnet metioned above
* two DNS entires as outlined in the official documentation

## Using the script

To use the script clone this repo and edit the group_vars/all.yml file.

To run the script run:

`ansible-playbook -i staging restricted_ipi.yml`

The playbook will take a while to run, especially if you are mirroring the Operator Catalog. In testing while mirroring the Operator Catalog the script can take up to 3 hours or more, due to the amount of data being mirrored. Successive runs of the script will skip these mirror sync steps if they have previously succeeded.

When the script completes sucesfuly you can run the openshift installer. BE SURE to use the local copy as this is configured to properly pull all required install images from your internal image registry.

`bin/opeshift-install create cluster --dir install-dir`

When the install completes if you mirrored the Operator Catalog follow these additional steps to enable it:

```
export KUBECONFIG=$(pwd)/install-dir/auth/kubeconfig
bin/oc patch OperatorHub cluster --type json \
    -p '[{"op": "add", "path": "/spec/disableAllDefaultSources", "value": true}]'
bin/oc apply -f ./olm/ocp45/redhat-operators-manifests
# note if you get an error from application of the manifest 
# edit the maifest and ensure that the "metadata.name" does not contain slashes (eg. \/)
bin/oc create -f ./olm/catalogsource.yaml
```

If all the above commands succeeded you can check the status with the following commands

```
bin/oc get pods -n openshift-marketplace
bin/oc get catalogsource -n openshift-marketplace
bin/oc get packagemanifest -n openshift-marketplace
```

**Congratulations you are the proud administrator of an OpenShift 4.5 Cluster on vSphere**