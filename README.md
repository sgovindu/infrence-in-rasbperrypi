# inference-in-rasbperrypi
This repo contains complete list of steps to install inference in raspberrypi 5.

Full disclosure: Some of the steps were obtained using google search. Consolidating the steps in a single place. Any changes I had to do will be explicitly mentioned.

## Pre install steps
1. Add ``cgroup_enable=cpuset cgroup_enable=memory cgroup_memory=1`` to end of the line ``/boot/firmware/cmdline.txt``
2. Disable swap ``sudo swapoff -a`` **Note:** This temporarily disables swap. Swap will get enabled after reboot

## Install k3s
**Note:** Installation of k3s is with cilium. I wanted to test with cilium as CNI because of its support of eBPF
1. Install k3s without cni
   ``
   sh curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="server" sh -s - \
      --flannel-backend=none \
      --disable-network-policy \
      --disable-kube-proxy \
      --disable traefik
  ``
3. After successful execution of this command, had to do the following changes
4. **Start changes**
5. kubectl get nodes resulted in permission error as ``K3S_KUBECONFIG_MODE="644"`` was not added during install. Explcitly changed the permissions using ``sudo chmod 644 /etc/rancher/k3s/k3s.yaml``
6. ``kubectl get nodes`` status was **Not Ready** as CNI was not installed. Its an expected status untill cilium is installed 
7. **End changes**
8. Install cilium 
   ``sh
CILIUM_CLI_VERSION=$(curl -s https://raw.githubusercontent.com/cilium/cilium-cli/main/stable.txt)
CLI_ARCH=arm64
curl -L --fail --remote-name-all \
    https://github.com/cilium/cilium-cli/releases/download/${CILIUM_CLI_VERSION}/cilium-linux-${CLI_ARCH}.tar.gz
sudo tar xzvfC cilium-linux-${CLI_ARCH}.tar.gz /usr/local/bin
rm cilium-linux-${CLI_ARCH}.tar.gz
``
9. ``cilium install --set kubeProxyReplacement=true`` installs cilium. **Note:** Plain ``cilium install`` will fail as OOTB CNI Fannel is not installed
10. Add ``export KUBECONFIG=/etc/rancher/k3s/k3s.yaml`` to ``$HOME/.bashrc``

## Add worker nodes (agents) to the control node
Configuring 2 RaspberryPI worker nodes. 
1. Extract the token from control-node using ``sudo cat /var/lib/rancher/k3s/server/node-token``
2. On the worker node, ``export TOKEN={output from step1}``
3. ``curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="agent --server https://<control_node_ip>:6443 --token $TOKEN" sh -``
4. Repeat steps 2 and 3 on the second worker node

Monitor cilium on the control-node using ``cilium status`` You should see cilium pods added to the newly added worker nodes. Use ``kubectl get pods -n kube-system -l k8s-app=cilium -o wide`` to check that three pods are running - one on control-node and 2 on worker nodes.

## Inference engines
There are two different inference engines that can be configured - llama.cpp from Meta and latest litertlm from Google.
Both of these inference engines use different model formats for execution. llama uses gguf models and litertlm uses litertlm models. Both these models can be downloaded from huggingface.

Both the docker files build the inference engines as a part of the image creation.

litertlm has dependency on rust and libvulkan1

### llama.cpp
llama/Dockerfile creates a docker image using llama.cpp

#### GGUF models download

From Huggingface search for < 4B models and filter on gguf. 
Select the model and download.

``curl -L -C - -O "https://huggingface.co/Qwen/Qwen2.5-3B-Instruct-GGUF/resolve/main/qwen2.5-3b-instruct-q4_k_m.gguf?download=true"``

``curl -L -C - -O "https://huggingface.co/unsloth/gemma-4-E2B-it-GGUF/resolve/main/gemma-4-E2B-it-UD-IQ2_M.gguf?download=true"``

### litertlm
litert-lm/Dockerfile creates a docker image that uses litert-lm.

#### litertlm models download

https://huggingface.co/litert-community contains models for litertlm. 

Models can be downloaded using

``curl -L -C - -O https://huggingface.co/litert-community/Qwen3-8B/resolve/main/qwen3_8b_mixed_int4.litertlm?download=true``

``curl -L -C - -O https://huggingface.co/litert-community/gemma-4-E2B-it-litert-lm/resolve/main/gemma-4-E2B-it.litertlm?download=true``
