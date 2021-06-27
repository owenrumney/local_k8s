# Notes

## Bridge network

Create a netplan bridge pointing to the physical interface

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
            interfaces: [ enp39s0 ]
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

## Host IP Tables

Some changes to host IP tables to route traffic from guest to internet correctly

```shell
sudo iptables -I FORWARD -i br0 -j ACCEPT
sudo iptables -I FORWARD -o br0 -j ACCEPT
```

### Running from scratch

Make sure that the vars in `ansible/k8s/group_vars/all.yaml` are correct, paying attention to paths and worker counts.

