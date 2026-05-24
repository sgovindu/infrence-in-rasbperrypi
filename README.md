# infrence-in-rasbperrypi
This repo contains complete list of steps to install inference in raspberrypi 5.

Full disclosure: All the steps were obtained using google search. Consolidating the steps in a single place. Any changes I had to do will be explicitly mentioned.

## Pre install steps
1. Add ``cgroup_enable=cpuset cgroup_enable=memory cgroup_memory=1`` to end of the line ``/boot/firmware/cmdline.txt``
2. Disable swap ``sudo swapoff -a`` **Note:** This temporarily disables swap. Swap will get enabled after reboot

## Install k3s
**Note:** Installation of k3s is with cilium. I wanted to test with cilium as CNI because of its support of eBPF
1. Install k3s without cni ``sh curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="server" sh -s - \
  --flannel-backend=none \
  --disable-network-policy \
  --disable-kube-proxy \
  --disable traefik``
2. After successful execution of this command, had to do the following changes
3. **Start changes**
4. kubectl get nodes resulted in permission error as ``K3S_KUBECONFIG_MODE="644"`` was not added during install. Explcitly changed the permissions using ``sudo chmod 644 /etc/rancher/k3s/k3s.yaml``
5. ``kubectl get nodes`` status was **Not Ready** as CNI was not installed. Its an expected status untill cilium is installed 
6. **End changes**
7. Install cilium ``CILIUM_CLI_VERSION=$(curl -s https://raw.githubusercontent.com/cilium/cilium-cli/main/stable.txt)
CLI_ARCH=arm64
curl -L --fail --remote-name-all https://github.com/cilium/cilium-cli/releases/download/${CILIUM_CLI_VERSION}/cilium-linux-${CLI_ARCH}.tar.gz
sudo tar xzvfC cilium-linux-${CLI_ARCH}.tar.gz /usr/local/bin
rm cilium-linux-${CLI_ARCH}.tar.gz``
8. ``cilium install --set kubeProxyReplacement=true`` installs cilium. **Note:** Plain ``cilium install`` will fail as OOTB CNI Fannel is not installed
9. Add ``export KUBECONFIG=/etc/rancher/k3s/k3s.yaml`` to ``$HOME/.bashrc``

