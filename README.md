# openshift-kvm

## Standard RHEL9 install

### Add Virtualization

dnf install qemu-kvm libvirt virt-install virt-viewer cockpit-machines wget
for drv in qemu network nodedev nwfilter secret storage interface; do systemctl start virt${drv}d{,-ro,-admin}.socket; done
virt-host-validate

#### Fix boot parameters - depending on CPU

##### Intel
intel_iommu=on

    grubby –update-kernel=ALL –args=”intel_iommu=on"

#### Enable Nested virtualization for Openshift Virtualization
/etc/modprobe.d/kvm.conf

		options kvm_intel nested=1

## Create VMS and Openshift Install artifacts

### Virtual Machines

    cd ~
    mkdir -p openshift/{vms,install,mirror}
    cd openshift/vms
    qemu-img create node1_os 120G
    qemu-img create node2_os 120G
    qemu-img create node3_os 120G
    virt-install --name node1 --memory 16384 --vcpus 8 --disk node1_os --import --os-variant rhl9 --noreboot
    virt-install --name node2 --memory 16384 --vcpus 8 --disk node2_os --import --os-variant rhl9 --noreboot
    virt-install --name node3 --memory 16384 --vcpus 8 --disk node3_os --import --os-variant rhl9 --noreboot

### Openshift - all under openshift/install
    
    cd ~/openshift/install
    // pull-secret.txt from console.redhat.com
    wget https://mirror.openshift.com/pub/openshift-v4/x86_64/clients/ocp/stable/openshift-install-linux.tar.gz
    wget https://mirror.openshift.com/pub/openshift-v4/x86_64/clients/ocp/stable/openshift-client-linux.tar.gz
    wget https://developers.redhat.com/content-gateway/rest/mirror/pub/openshift-v4/clients/mirror-registry/latest/mirror-registry.tar.gz
    ls *tar.gz | xargs -n 1 tar -zxvf
    mkdir ~/bin
    mv oc kubectl openshift-install ~/bin
    
    
            
