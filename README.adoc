= openshift-kvm

== Standard RHEL9 install

=== Create a Openshift User with admin priviledges

=== Add Virtualization

dnf install qemu-kvm libvirt virt-install virt-viewer cockpit-machines wget podman 
for drv in qemu network nodedev nwfilter secret storage interface; do systemctl start virt${drv}d{,-ro,-admin}.socket; done
virt-host-validate

=== Fix boot parameters - depending on CPU

=== Intel
intel_iommu=on

    grubby –update-kernel=ALL –args=”intel_iommu=on"

=== Enable Nested virtualization for Openshift Virtualization
/etc/modprobe.d/kvm.conf

		options kvm_intel nested=1

== Environment

=== Add env variables to keep track of settings

[source]
----
cat <<EOF >> .bashrc
QUAY_HOSTNAME=$(hostname -f)
QUAY_USER=quay
QUAY_PASSWORD=quayquay
EOF
----

## Create VMS and Openshift Install artifacts

### Virtual Machines

    cd ~
    mkdir -p openshift/{vms,install,mirror,downloads}
    cd openshift/vms
    qemu-img create node1_os 120G
    qemu-img create node2_os 120G
    qemu-img create node3_os 120G
    virt-install --name node1 --memory 16384 --vcpus 8 --disk node1_os --import --os-variant rhl9 --noreboot
    virt-install --name node2 --memory 16384 --vcpus 8 --disk node2_os --import --os-variant rhl9 --noreboot
    virt-install --name node3 --memory 16384 --vcpus 8 --disk node3_os --import --os-variant rhl9 --noreboot

### Openshift - binaries
    
    cd ~/openshift/downloads
    wget https://mirror.openshift.com/pub/openshift-v4/x86_64/clients/ocp/stable/openshift-install-linux.tar.gz
    wget https://mirror.openshift.com/pub/openshift-v4/x86_64/clients/ocp/stable/openshift-client-linux.tar.gz
    ls *tar.gz | xargs -n 1 tar -zxvf
    mkdir ~/bin
    mv oc kubectl openshift-install ~/bin
    wget https://developers.redhat.com/content-gateway/rest/mirror/pub/openshift-v4/clients/mirror-registry/latest/mirror-registry.tar.gz

## Certificates and pull-secret
    
    cd ~/install
    // pull-secret.txt from console.redhat.com/openshift/downloads
    // Certificate rhel9 host (or wildcard for rhel9 host domain) cert/key
    // reg.pem
    // regkey.pem
    
    
## Setup Mirror
    

### Mirror Registry

    cd ~/openshift/mirror
    tar zxvf downloads/mirror-registry.tar.gz
    ssh-keygen
    ssh-copy-id $QUAY_HOSTNAME
    sudo ./mirror-registry install --initUser $QUAY_USER --initPassword $QUAY_PASSWORD --quayHostname ${QUAY_HOSTNAME} --sslCert ../install/reg.pem --sslKey ../install/regkey.pem
    sudo --add-port=8443/tcp --zone=public --permanent
    sudo firewall-cmd --reload

     
    
            
