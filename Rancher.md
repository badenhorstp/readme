 
## 2. Generate SSH Keypair
To start out, we will generate a new SSH Keypair and place this keypair on the node we will install Kubernetes for Rancher onto. As we will be using the rancher01 node to run Rancher + Kubernetes, we will simply copy the public key into the authorized_keys file of this node.

The following command will generate the keypair and copy it into the file.
```bash
ssh-keygen -b 2048 -t rsa -f \
/home/ubuntu/.ssh/id_rsa -N ""
cat /home/ubuntu/.ssh/id_rsa.pub \
>> /home/ubuntu/.ssh/authorized_keys
```

## 3. Download RKE
Rancher Kubernetes Engine (RKE) is an extremely simple, lightning fast Kubernetes installer that works everywhere.

In this step, we will download the RKE CLI
```bash
sudo wget -O /usr/local/bin/rke \
https://github.com/rancher/rke/releases/download/v1.3.0/rke_linux-amd64
```
In order to execute RKE, we need to mark it as executable:
```bash
sudo chmod +x /usr/local/bin/rke

Next, let's validate that RKE is installed properly:
```bash
rke -v
```
You should have an output similar to:

rke version v1.3.0

If you receive the output as expected, you can continue on to the next step.

## 4. Install Kubectl
In order to interact with our Kubernetes cluster after we install it using rke, we need to install kubectl.

The following command will add an apt repository and install kubectl:
```bash
curl -LO "https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl
```

After the apt-get install -y kubectl command finishes, we can test kubectl and make sure it is properly installed.
```bash
kubectl version --client --short
```

## 5. Install Helm
Helm is a very popular package manager for Kubernetes. It is used as the installation tool for Rancher when deploying Rancher onto a Kubernetes cluster. 
In order to download Helm, we need to download the Helm install script, mark it as executable and then run it. Finally, we will clean up the helm install script.
```bash
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
rm -f get_helm.sh
```

After a successful installation of Helm, we should check our installation to ensure that we are ready to install Rancher.
```bash
helm version --client --short
```

## 6. Create a rancher-cluster.yaml file
RKE CLI uses a YAML-formatted file to describe the configuration of our cluster. The following command will heredoc write the corresponding file onto your rancher01 node, so that RKE will be able to install Kubernetes.

RKE uses SSH tunneling, which is why we generated the keypair in the first part of this scenario.
```bash
cat << EOF > rancher-cluster.yml
nodes:
  - address: 34.228.217.202
    internal_address: 172.31.43.130
    user: ubuntu
    role: [controlplane,etcd,worker]
addon_job_timeout: 120
kubernetes_version: "v1.21.4-rancher1-1"
EOF
```

## 7. Run RKE Up
We are now ready to run rke up to install Kubernetes onto our node.

The following command will run rke up which will install Kubernetes onto our node:
```bash
rke up --config rancher-cluster.yml
```

## 8. Testing your Cluster
RKE will have generated two important files:
* kube_config_rancher-cluster.yml
* rancher-cluster.rkestate

in addition to your
* rancher-cluster.yml

All of these files are extremely important for future maintenance of our cluster. When running rke on your own machines to install Kubernetes/Rancher, you must make sure you have current copies of all 3 files otherwise you can run into errors when running rke up.

The kube_config_rancher-cluster.yml file will contain a kube-admin kubernetes context that can be used to interact with your Kubernetes cluster that you've installed Rancher on.

We can soft symlink the kube_config_rancher-cluster.yml file to our /home/ubuntu/.kube/config file so that kubectl can interact with our cluster:
```bash
mkdir -p /home/ubuntu/.kube
ln -s /home/ubuntu/kube_config_rancher-cluster.yml /home/ubuntu/.kube/config
```

In order to test that we can properly interact with our cluster, we can execute two commands:
```bash
kubectl get nodes
```
```bash
kubectl get pods --all-namespaces
```

## 9. Install Cert Manager
cert-manager is a Kubernetes add-on to automate the management and issuance of TLS certificates from various issuing sources.

The following set of steps will install cert-manager which will be used to manage the TLS certificates for Rancher.

The following command will apply the cert-manager custom resource definitions as well as label the namespace that cert-manager runs in to disable validation:
```bash
helm repo add jetstack https://charts.jetstack.io
helm repo update
helm install \
  cert-manager --namespace cert-manager \
  --create-namespace \
  --version v1.5.3 \
  --set installCRDs=true \
  jetstack/cert-manager
```

Verify cert-manager installed correctly:
```bash
kubectl get pods --namespace cert-manager
kubectl rollout status -n cert-manager deploy/cert-manager
kubectl rollout status -n cert-manager deploy/cert-manager-cainjector
kubectl rollout status -n cert-manager deploy/cert-manager-webhook
```

## 10. Install Rancher
We will now install Rancher in HA mode onto our rancher01 Kubernetes cluster. 
The following command will add rancher-latest as a helm repository and update our local repository cache:
```bash
helm repo add rancher-stable https://releases.rancher.com/server-charts/stable
helm repo update
```

Create cattle namespace:
```bash
kubectl create namespace cattle-system
```

Finally, we can install Rancher using our helm install command:
```bash
helm install rancher rancher-stable/rancher \
  --namespace cattle-system \
  --set hostname=rancher.34.228.217.202.on.hobbyfarm.io \
  --set replicas=1 \
  --version 2.5.9
```

Wait for Rancher to be rolled out:
```bash
kubectl -n cattle-system rollout status deploy/rancher
```
