# Notes

## Bridge network

Create a netplan bridge pointing to the physical interface, this will go into `/etc/netplan/01_kvm_bridge.yaml`

```yaml
network:
    version: 2
    renderer: networkd
    ethernets:
        enp39s0:
            dhcp4: false
            dhcp6: false
    bridges:
        bro0:
            interfaces: [ enp39s0 ] # the interface on your machine
            addresses: [ 192.168.0.100/24 ]
            gateway4: 192.168.0.1
            mtu: 1500
            nameservers:
                addresses: [ 8.8.8.8, 8.8.4.4 ]
            parameters:
                stp: true
                forward-delay: 4
            dhcp4: no
            dhcp6: no
```

Now generate with `sudo netplan generate`

Now apply with `sudo netplan apply`

## Host IP Tables

Some changes to host IP tables to route traffic from guest to internet correctly

```bash
sudo iptables -I FORWARD -i br0 -j ACCEPT
sudo iptables -I FORWARD -o br0 -j ACCEPT
```

### Downloading the image

Run `images/get_focal_image.sh` to download the ubuntu image locally


### Running from scratch

Make sure that the vars in `ansible/k8s/group_vars/all.yaml` are correct, paying attention to paths and worker counts.

```bash
cd ansible/k8s

export ANSIBLE_PASSWORD=<yourpassword>

ansible-playbook -i inventories/hosts.ini install_from_scratch.yaml
```

### Kubectl

As part of the process, the kube config will be pulled back to your machine and put into the folder `~/.kube/virtconfig/<masterHostName>/etc/kubernetes/admin.conf` to protect any existing config you have. To use the virt cluster, `export KUBECONFIG `

```bash
export KUBECONFIG=~/.kube/virtconfig/192.168.0.150/etc/kubernetes/admin.conf
```

Now you can use `kubectl` to interact with the cluster

### Destroying the cluster

```bash
virsh destroy master-1 && virsh undefine master-1

for i in {1-3}; do virsh destroy worker-$i && virsh undefine worker-$i; done
```