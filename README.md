# kubernetes-cluster

## Fedora Image 
[Fedora.ISO](https://builds.coreos.fedoraproject.org/prod/streams/stable/builds/43.20251024.3.0/x86_64/fedora-coreos-43.20251024.3.0-live-iso.x86_64.iso)

## Create VMs
``` 
# virt-install \
-n test \
--description="Cloud Kubernetes test" \
--os-variant=fedora-unknown \
--ram=4096 \
--vcpus=2 \
--disk path=/var/lib/libvirt/images/test.img,bus=virtio,size=20 \
--channel qemu-vdagent,target.type=virtio,target.name=org.libguestfs.channel.0 \
--video qxl \
--cdrom /home/cpqd/Downloads/Fedora-Server-netinst-x86_64-43-1.6.iso \
--network bridge:virbr0
```

@@ Update OS and prepare kernel
```
dnf update -y
sudo dnf install gpm spice-vdagent -y
sudo systemctl stop swap-create@zram0
sudo dnf remove zram-generator-defaults -y
sudo systemctl disable --now firewalld
sudo dnf install iptables iproute-tc -y
sudo cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF
sudo modprobe overlay
sudo modprobe br_netfilter
sudo cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF
sudo sysctl --system
sysctl net.bridge.bridge-nf-call-iptables net.bridge.bridge-nf-call-ip6tables net.ipv4.ip_forward
sudo dnf install cri-o1.31 containernetworking-plugins -y
sudo dnf install kubernetes1.34 kubernetes1.34-kubeadm kubernetes1.34-client -y
sudo systemctl enable --now crio
sudo kubeadm config images pull
sudo systemctl enable --now kubelet
sudo reboot now

```

@@ Init kluster
```
sudo kubeadm init --pod-network-cidr=10.244.0.0/16
```

## Copy kubeconfig
```
mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config
``
