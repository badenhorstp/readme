# Lab Session One

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

# Lab Session Two
## 1. Welcome to Phase II
In this lab we will have two nodes, a Rancher server node and a Kubernetes node. The Rancher server will be deployed as part of this hands on lab.

Once Rancher is deployed we'll need to login to our newly setup Rancher UI.

In the next step we will deploy a k3s Kubernetes cluster where Rancher will be deployed.

## 2. Install Kubernetes with K3s
K3s is the easiest and simplest way to get started with production-grade Kubernetes. It's a fully-conformant CNCF Kubernetes distribution, and further information you can read more at the project's official website.

Running the following command to install k3s onto your Rancher Server VM:
```bash
curl -sfL https://get.k3s.io | INSTALL_K3S_VERSION=v1.21.3+k3s1 sh -
```
This will take about 30 seconds to complete. To verify that it's working, run the following command:
```bash
sudo k3s kubectl get node
```
We now have a fully-functioning Kubernetes cluster that we can install Rancher into!

## 3. Setting Up Rancher
We are going to use Helm v3 to install Rancher Server into our cluster, so we need to install that first:
```bash
sudo wget -O helm.tar.gz \
https://get.helm.sh/helm-v3.4.1-linux-amd64.tar.gz
sudo tar -zxf helm.tar.gz
sudo mv linux-amd64/helm /usr/local/bin/helm
sudo chmod +x /usr/local/bin/helm
sudo rm -rf linux-amd64 helm.tar.gz
```
 
We'll also drop in place the necessary configuration files for kubectl to work:
```bash
sudo cp /etc/rancher/k3s/k3s.yaml ~/.kube/config
sudo chown -R ubuntu:ubuntu ~/.kube
export KUBECONFIG=~/.kube/config
```
After a successful installation of Helm, let's check to make sure we are ready to install Rancher:
```bash
helm version --short
```
We then need to install some prerequisites for Rancher, cert-manager is required to allow Rancher to generate its self signed certs. If you are installing using your own certificates this can be omitted. More information is available at Rancher Docs
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
Once the Helm chart has installed, you can monitor the rollout status of both cert-manager and cert-manager-webhook:
```bash
kubectl -n cert-manager rollout status deploy/cert-manager
```
You should eventually receive output similar to:

Waiting for deployment "cert-manager" rollout to finish:
0 of 1 updated replicas are available...

deployment "cert-manager" successfully rolled out
To check the status of the cert-manager-webhook, we can run a similar command:
```bash
kubectl -n cert-manager rollout status deploy/cert-manager-webhook
```
You should eventually receive output similar to:

Waiting for deployment "cert-manager-webhook" rollout to finish:
0 of 1 updated replicas are available...

deployment "cert-manager-webhook" successfully rolled out
Next we will install Rancher:

Install Rancher
```bash
helm repo add rancher-latest https://releases.rancher.com/server-charts/latest
helm repo update
helm install rancher rancher-latest/rancher --namespace cattle-system \
  --create-namespace \
  --version v2.5.9 \
  --set hostname=rancher.3.88.254.150.on.hobbyfarm.io \
  --set replicas=1
```
Before we access Rancher, we need to make sure that cert-manager has signed a certificate in order to make sure our connection to Rancher does not get interrupted. The following bash script will check for the certificate we are looking for.
```bash
while true; do 
  curl -kv https://rancher.3.88.254.150.on.hobbyfarm.io 2>&1 | grep -q "200"
  if [ $? != 0 ]; then 
    echo "Rancher isn't ready yet"
    sleep 5
    continue
  fi
  break
done
echo "Rancher is Ready"
```
This can take a few minutes, when it shows that "Rancher is Ready" you can proceed to the next step.

## 5. Setting up a Rancher-managed Kubernetes cluster
In this step, we will be creating a Kubernetes Lab environment within Rancher. Normally, in a production case, you would create a Kubernetes Cluster with multiple nodes; however, with this lab environment, we will only be using one virtual machine for the cluster.

Hover over the top left dropdown, then click Global.
Click Add Cluster.
The current context is shown in the upper left, and should say 'Global'.
Note the multiple types of Kubernetes cluster Rancher supports. We will be using Custom for this lab, but there are a lot of possibilities with Rancher.
Click on the From existing nodes (Custom) Cluster box.
Enter a name in the Cluster Name box and click Next at the bottom.
Make sure the boxes etcd, Control Plane, and Worker are all ticked.
Click Show advanced options to the bottom right of the Worker checkbox.
Enter the Public Address (54.196.70.105) and Internal Address (172.31.42.166).
IMPORTANT: It is VERY important that you use the correct External and Internal addresses from the node01 machine for this step, and run it on the correct machine. Failure to do this will cause the future steps to fail.
Click the clipboard to Copy to Clipboard the docker run command.
Click Done.
NOTE: You can find the docker command again by editing the cluster in the UI if needed.

## 6. Bootstrapping a node in your Kubernetes Cluster
IMPORTANT NOTE: Make sure you have selected the node01 tab in HobbyFarm in the window to the right. If you run this command on the rancher node you will cause problems for your scenario session.

Take the copied docker command and run it on node01.
Within the Rancher UI click on <YOUR_CLUSTER_NAME> which is the name you entered during cluster creation.
You can watch the state of the cluster as your Kubernetes node node01 registers with Rancher here as well as the Nodes tab.
Your cluster state on the Global page will change to Active.
Once your cluster has gone to Active you can click on it and start exploring.

## 7. Setup User Governance
First we are going to setup GitHub Authentication so we can leverage external identities within Kubernetes.

From the Global context, select the menu for Security then select Authentication.

Here you will be presented with a series of tiles representing the different auth providers Rancher supports.

Select the GitHub icon. Now in the instructions under step 1, select "click here" to open the GitHub applications page. Here you will register a new application. Fill out the form according to the instructions.

After you fill out the form, you should get a pop-up asking you to authenticate with GitHub. Once you authorize with GH, you will be redirected to a new page in Rancher where you can manage GH settings.

In this GitHub settings page, under the section "Site Access", change the value to "Allow any valid users". Click Save.

## 8. Create a Project
Now go back to our cluster in the Rancher UI (you can select this from the upper left hand menu).

Once in the cluster context, select "Projects / Namespaces" from the top menu bar. From the projects page select "Add Project".

Here you can specify which users can access this project and with what permissions. Add the GitHub user yankcrime to your project as a Read Only user. You should now have two users in the project, your own GitHub handle as the Owner, and a secondary user that is Read Only.

In order to make sure that users have a baseline definition of resource requests limits, we're going to set some defaults. Select limits and set a default limit of 4GB memory, 500 milicpu. Click Create.

Next we'll also want to create a Namespace within our project. From the projects page, click Add Namespace next to the project title. Give the namespace the same name as the project. Click Create.

## 9. Setup PSPs
Next we're going to enforce a Pod Security Policy. This ensures that pods will only be allowed into the cluster if they comply with our security standards.

First, we'll need to enable PSPs at the Cluster level. From the "Global" section, select the options menu (three dots) on the right side of the page for the cluster we created. Click "Edit". Scroll down to the "Advanced Options" section, and toggle "Pod Security Policy Support" to "Enabled". Also make sure the default PSP is unrestricted. Click Save.

Now go back to the "Projects/Namespaces" section, and edit the project we created. In our project, select the restricted PSP from the drop down menu to enable this. Click Save.

Finally let's try to deploy an app that doesn't comply with the PSP we just enforced and see what happens. Navigate to the project we created by selecting the cluster from the top left menu, and then hovering over to the project name to the right of it. Then go "Resources" -> "Workloads". From here click the button in the top right that says "Import YAML", and copy the following into the box and click Import:
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-insecure-app
spec:
  replicas: 1
  selector:
    matchLabels:
      workload.user.cattle.io/workloadselector: deployment-default-my-insecure-app
  template:
    metadata:
      labels:
        workload.user.cattle.io/workloadselector: deployment-default-my-insecure-app
    spec:
      containers:
      - image: nginx
        name: my-insecure-app
        securityContext:
          privileged: true
```

# Lab Session Three
## 2. Install Rancher in HA mode
First, we'll need to setup Rancher in HA mode. This means that the Rancher Management Server will be installed on top of Kubernetes.

K3s is the easiest and simplest way to get started with production-grade Kubernetes. It's a fully-conformant CNCF Kubernetes distribution, and further information you can read more at the project's official website.

Running the following command to install k3s onto your Rancher Server VM:
```bash
curl -sfL https://get.k3s.io | INSTALL_K3S_VERSION=v1.21.3+k3s1 sh -
```
 
This will take about 30 seconds to complete. To verify that it's working, run the following command:
```bash
sudo k3s kubectl get node
```
We now have a fully-functioning Kubernetes cluster that we can install Rancher into!

We are going to use Helm v3 to install Rancher Server into our cluster, so we need to install that first:
```bash
sudo wget -O helm.tar.gz \
https://get.helm.sh/helm-v3.4.1-linux-amd64.tar.gz
sudo tar -zxf helm.tar.gz
sudo mv linux-amd64/helm /usr/local/bin/helm
sudo chmod +x /usr/local/bin/helm
sudo rm -rf linux-amd64 helm.tar.gz
```
We'll also drop in place the necessary configuration files for kubectl to work:
```bash
sudo cp /etc/rancher/k3s/k3s.yaml ~/.kube/config
sudo chown -R ubuntu:ubuntu ~/.kube
export KUBECONFIG=~/.kube/config
```
After a successful installation of Helm, let's check to make sure we are ready to install Rancher:
```bash
helm version --short
```
We then need to install some prerequisites for Rancher, cert-manager is required to allow Rancher to generate its self signed certs. If you are installing using your own certificates this can be omitted. More information is available at Rancher Docs
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
Once the Helm chart has installed, you can monitor the rollout status of both cert-manager and cert-manager-webhook:
```bash
kubectl -n cert-manager rollout status deploy/cert-manager
```
You should eventually receive output similar to:
```
Waiting for deployment "cert-manager" rollout to finish:
0 of 1 updated replicas are available...

deployment "cert-manager" successfully rolled out
```
To check the status of the cert-manager-webhook, we can run a similar command:
```bash
kubectl -n cert-manager rollout status deploy/cert-manager-webhook
```
You should eventually receive output similar to:
```
Waiting for deployment "cert-manager-webhook" rollout to finish:
0 of 1 updated replicas are available...

deployment "cert-manager-webhook" successfully rolled out
```
Next we will install Rancher:

Install Rancher
```bash
helm repo add rancher-latest https://releases.rancher.com/server-charts/latest
helm repo update
helm install rancher rancher-latest/rancher --namespace cattle-system \
  --create-namespace \
  --version v2.5.9 \
  --set hostname=rancher.54.92.191.19.on.hobbyfarm.io \
  --set replicas=1
```
Before we access Rancher, we need to make sure that cert-manager has signed a certificate in order to make sure our connection to Rancher does not get interrupted. The following bash script will check for the certificate we are looking for.
```bash
while true; do 
  curl -kv https://rancher.54.92.191.19.on.hobbyfarm.io 2>&1 | grep -q "200"
  if [ $? != 0 ]; then 
    echo "Rancher isn't ready yet"
    sleep 5
    continue
  fi
  break
done
echo "Rancher is Ready"
```
This can take a few minutes, when it shows that "Rancher is Ready" you can proceed to the next step.

## 3. Setup A Kubernetes Cluster
In this step, we will be creating a Kubernetes Lab environment within Rancher. Normally, in a production case, you would create a Kubernetes Cluster with multiple nodes; however, with this lab environment, we will only be using one virtual machine for the cluster.

First, we'll need to login to our newly setup Rancher UI.

Select the server rancher01, on the right and take note of its public address. This will be the address we use to connect to the Rancher UI. For convenience you should be able to use this URL: https://rancher.54.92.191.19.on.hobbyfarm.io

Once you open that URL in a browser, set a default password and confirm the URL that should be used by the server after that.

Hover over the top left dropdown, then click Global.
Click Add Cluster.
The current context is shown in the upper left, and should say 'Global'.
Note the multiple types of Kubernetes cluster Rancher supports. We will be using Custom for this lab, but there are a lot of possibilities with Rancher.
Click on the From existing nodes (Custom) Cluster box.
Enter a name in the Cluster Name box and click Next at the bottom.
Make sure the boxes etcd, Control Plane, and Worker are all ticked.
Click Show advanced options to the bottom right of the Worker checkbox.
Enter the Public Address (35.173.183.83) and Internal Address (172.31.43.156).
IMPORTANT: It is VERY important that you use the correct External and Internal addresses from the node01 machine for this step, and run it on the correct machine. Failure to do this will cause the future steps to fail.
Click the clipboard to Copy to Clipboard the docker run command.
Click Done.
NOTE: You can find the docker command again by editing the cluster in the UI if needed.

## 4. Bootstrapping a Node in your Kubernetes Cluster
IMPORTANT NOTE: Make sure you have selected the node01 tab in HobbyFarm in the window to the right. If you run this command on Rancher01 you will cause problems for your scenario session.

Take the copied docker command and run it on node01.
Within the Rancher UI click on <YOUR_CLUSTER_NAME> which is the name you entered during cluster creation.
You can watch the state of the cluster as your Kubernetes node node01 registers with Rancher here as well as the Nodes tab.
Your cluster state on the Global page will change to Active.
Once your cluster has gone to Active you can click on it and start exploring.

## 5. Setup Persistent Storage using Longhorn
From the Rancher UI, within the default project, navigate to the Apps section of the UI. From here, click on Launch, and search for the chart called Longhorn. Click on the icon.

Leave the default settings as is, then scroll to the bottom and click Launch.

Give the chart some time to deploy. When it finishes it will have registered a new storage class. Navigate to the storage section to view the storage class details.

Now try launching an app with persistence enabled. From the Apps section, click Launch, then select wordpress. The wordpress chart will have a number of options we can configure. We just want to change the settings involving persistent storage. Find the options called WordPress Persistent Volume Enabled and MariaDB Persistent Volume Enabled and make these True. Then click Launch.

That's it. Now your wordpress chart should create stateful sets with accompanying PVs. You can click on the app in the Apps page and see the PVs and related objects being created.

## 6. Setup a Helm Catalog
First, we need to setup a git repo. On rancher01:
```bash
cd 
mkdir my_charts
cd my_charts
git init
```
Now we need to install the Helm CLI:
```bash
sudo wget -O helm.tar.gz \
https://get.helm.sh/helm-v2.14.3-linux-amd64.tar.gz
sudo tar -zxf helm.tar.gz
sudo mv linux-amd64/helm /usr/local/bin/helm
sudo chmod +x /usr/local/bin/helm
sudo rm -rf linux-amd64
sudo rm -f helm.tar.gz
```
Next we want to create a Helm Chart skeleton from which we can build our chart:
```
cd 
cd my_charts
helm create hello-world
```
To customize the chart to our uses, we can update the values.yml file like so:
```bash
cat << EOF > ~/my_charts/hello-world/values.yaml
replicaCount: 1

image:
  repository: tutum/hello-world
  tag: latest
  pullPolicy: IfNotPresent

imagePullSecrets: []
nameOverride: ""
fullnameOverride: ""

service:
  type: ClusterIP
  port: 80

ingress:
  enabled: true
  annotations: {}
    # kubernetes.io/ingress.class: nginx
    # kubernetes.io/tls-acme: "true"
  hosts:
    - host: on.hobbyfarm.io
      paths: ["/"]

  tls: []

resources: {}
nodeSelector: {}
tolerations: []
affinity: {}
EOF
```
Next we need to setup a simple git server to facilitate exposing this chart to Rancher. This will create a "catalog" from which Rancher can fetch Helm charts a user would like to deploy. In a real-world scenario, we'd likely host these repos on a git server or some internal Helm repository.
```bash
cd ~/my_charts/
git config --global user.email "you@example.com"
git config --global user.name "Your Name"
git add --all
git commit -a -m "first commit" 
git config --global alias.quickserve "daemon --verbose --export-all --base-path=.git --reuseaddr --strict-paths .git/"
git quickserve
```
Now in the Rancher UI go to "Add catalogs" and add the URL of this server:
```
git://172.31.44.79/
```
Your catalog should now be imported into the Apps section of the UI. Now simply go back to the Apps tab and click Launch. Then search for your app (hello-world) and click on the icon. Scroll to the bottom and click Launch.

# Lab Session Four
## Step 1 - Install Kubernetes (K3s)
K3s is simple to get started with - further information can be found at the official website.

Running the following command to install k3s onto your Rancher Server VM:

curl -sfL https://get.k3s.io | INSTALL_K3S_VERSION=v1.19.5+k3s2 sh -
 Click to run on developer
This will take about 30 seconds to complete. To verify that it's working, run the following command:

sudo k3s kubectl get node
 Click to run on developer
We now have a fully-functioning Kubernetes cluster that we can install Rancher into! 🎉

## Step 2 - Install Rancher Server
We are going to use Helm v3 to install Rancher Server into our cluster:

sudo snap install helm --classic
sudo cp /etc/rancher/k3s/k3s.yaml ~/.kube/config
sudo chown -R ubuntu:ubuntu ~/.kube
export KUBECONFIG=~/.kube/config
 Click to run on developer
After a successful installation of Helm, let's check to make sure we are ready to install Rancher:

helm version --short
 Click to run on developer
We then need to install some prerequisites for Rancher, cert-manager is required to allow Rancher to generate its self signed certs. If you are installing using your own certificates this can be omitted. More information is available at Rancher Docs

helm repo add jetstack https://charts.jetstack.io
helm repo update
helm install \
  cert-manager --namespace cert-manager \
  --create-namespace \
  --version v1.4.0 \
  --set installCRDs=true \
  jetstack/cert-manager
 Click to run on developer
Once the Helm chart has installed, you can monitor the rollout status of both cert-manager and cert-manager-webhook:

kubectl -n cert-manager rollout status deploy/cert-manager
 Click to run on developer
You should eventually receive output similar to:

Waiting for deployment "cert-manager" rollout to finish: 0 of 1 updated replicas are available...

deployment "cert-manager" successfully rolled out
To check the status of the cert-manager-webhook, we can run a similar command:

kubectl -n cert-manager rollout status deploy/cert-manager-webhook
 Click to run on developer
You should eventually receive output similar to:

Waiting for deployment "cert-manager-webhook" rollout to finish: 0 of 1 updated replicas are available...

deployment "cert-manager-webhook" successfully rolled out
Next we will install Rancher:

Install Rancher

helm repo add rancher-latest https://releases.rancher.com/server-charts/latest
helm repo update
helm install rancher rancher-latest/rancher --namespace cattle-system \
  --create-namespace \
  --set hostname=rancher.3.69.238.183.nip.io \
  --version=2.5.11 \
  --set replicas=1
 Click to run on developer
Before we access Rancher, we need to make sure that cert-manager has signed a certificate in order to make sure our connection to Rancher does not get interrupted. The following bash script will check for the certificate we are looking for.

while true; do
  curl -kv --insecure https://rancher.3.69.238.183.nip.io 2>&1 | grep -q "200"
  if [ $? != 0 ]; then
    echo "Rancher isn't ready yet"
    sleep 5
    continue
  fi
  break
done
echo "Rancher is Ready"
 Click to run on developer
This can take a few minutes, when it shows that "Rancher is Ready" you can proceed to the next step.

## Step 3 - Accessing Rancher
Access Rancher Server at https://rancher.3.69.238.183.nip.io
Enter a password for the default admin user when prompted.
When prompted, the Rancher Server URL should be rancher.3.69.238.183.nip.io, which is the hostname you used to access the server.
You will see the Rancher UI, with the local cluster in it. The local cluster is the cluster where Rancher itself runs, and typically should not be used for deploying your demo workloads. However, for the purposes of this lab, we're going to use this cluster to install OPA Gatekeeper and demonstrate basic functionality.

## Step 4 - Installing Open Policy Agent Gatekeeper
With Rancher installed, we can set about installing Open Policy Agent Gatekeeper.

Click on the 'Explorer' button for our local cluster
Click on 'Apps' in the toolbar at the top
Select 'OPA Gatekeeper', scroll to the bottom, and click 'Install' to deploy Gatekeeper with the default options
Wait until rancher-gatekeeper is marked as Deployed
Once OPA Gatekeeper has been installed to our cluster, we can begin to define policies (ConstraintTemplates) within our cluster, and enable them via Constraints.

## Step 5 - Deploy privileged container
First of all, let's see what happens when we deploy a privileged container with the default behaviour and no policy in place preventing such an action.

In the Rancher UI, click on the dropdown menu in the top left-hand corner and select 'Cluster Explorer'.
From the menu on the left, click 'Pods' under the 'Workload' section
Click on the 'Create from YAML' button and paste in the following Pod definition, overwriting the existing example template:
apiVersion: v1
kind: Pod
metadata:
  name: nginx-test
  namespace: default
  labels:
    app: nginx-privileged
spec:
  hostPID: true
  containers:
  - name: nginx
    image: nginx
    securityContext:
      privileged: true
Click on 'Create'.
After a few seconds, you'll see that the Pod with a privileged container has been created successfully.

## Step 6 - Installing and enabling an example Policy
With the Rancher OPA Gatekeeper App installed to our cluster, we can install OPA policies and then have them applied and enforced.

In this step, we're going to define a policy that denies the creation of privileged containers. The policy will tell OPA Gatekeeper to examine the spec for for any Pod and look for a securityContext section along with the privileged flag. If that's set to true, then Gatekeeper will return an error back to the client via the Kubernetes API Server.

Let's create the ConstraintTemplate object for this policy:

kubectl apply -f - << EOF
apiVersion: templates.gatekeeper.sh/v1beta1
kind: ConstraintTemplate
metadata:
  name: k8spspprivilegedcontainer
  annotations:
    description: Controls running of privileged containers.
spec:
  crd:
    spec:
      names:
        kind: K8sPSPPrivilegedContainer
  targets:
    - target: admission.k8s.gatekeeper.sh
      rego: |
        package k8spspprivileged

        violation[{"msg": msg, "details": {}}] {
            c := input_containers[_]
            c.securityContext.privileged
            msg := sprintf("Privileged container is not allowed: %v, securityContext: %v", [c.name, c.securityContext])
        }

        input_containers[c] {
            c := input.review.object.spec.containers[_]
        }

        input_containers[c] {
            c := input.review.object.spec.initContainers[_]
        }
EOF
 Click to run on developer
Then we enable its enforcement for resources created under the Pod API Group:

kubectl apply -f - << EOF
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: K8sPSPPrivilegedContainer
metadata:
  name: psp-privileged-container
spec:
  match:
    kinds:
      - apiGroups: [""]
        kinds: ["Pod"]
EOF
 Click to run on developer
With the both these resources defined, we should now be able to test OPA Gatekeeper to make sure it's enforcing our policy.

## Step 7 - Test OPA Gatekeeper
With our ConstraintTemplate defined and a Constraint in place targeting the Pod API Group, we can test what happens when a user attempts to create a privileged Pod.

Back in the Rancher UI, click on the dropdown menu in the top left-hand corner and select 'Cluster Explorer'.
From the menu on the left, click 'Pods' under the 'Workload' section
Click on the 'Create from YAML' button and paste in the following Pod definition, overwriting the existing example template:
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  namespace: default
  labels:
    app: nginx-privileged
spec:
  hostPID: true
  containers:
  - name: nginx
    image: nginx
    securityContext:
      privileged: true
Click on 'Create'.
You should see a red error box appear stating the specific reason as defined in our ConstraintTemplate. If so, congratulations! You've successfully installed OPA Gatekeeper to a Kubernetes cluster, created a custom policy, and ensured its enforcement.

Feel free to explore the 'OPA Gatekeeper' workspace via the Rancher UI. You'll notice that there are two additional templates installed by default - k8sallowedrepos and k8srequiredlabels. Take a look at these to understand the policy restrictions they'd potentially impose.

## Step 8 - Second policy example
In this step, we're going to define a policy that ensures that namespaces have the required label. The policy will tell OPA Gatekeeper to examine the metadata for any Pod and look for a manager label. The setting for the label can be anything, it the policy just checks for the existence of the label.

Let's create the ConstraintTemplate object for this policy:

kubectl apply -f - << EOF
apiVersion: templates.gatekeeper.sh/v1beta1
kind: ConstraintTemplate
metadata:
  name: checklabeltemplate
spec:
  crd:
    spec:
      names:
        kind: CheckLabelTemplate
  targets:
    - target: admission.k8s.gatekeeper.sh
      rego: |
        package checklabels
        violation[{"msg": msg}] {
          labels := { label | input.review.object.metadata.labels[label] }
          required := { label | label := "manager"}
          missing := required - labels
          count(missing) > 0
          msg := sprintf("Missing \"manager\" label, labels provided: %v", [labels])
        }
EOF
 Click to run on developer
Then we enable its enforcement for resources by creating a constraint:

kubectl apply -f - << EOF
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: CheckLabelTemplate
metadata:
  name: check-for-label
spec:
  match:
    kinds:
      - apiGroups: [""]
        kinds: ["Namespace", Pod"]
EOF
 Click to run on developer
With the both these resources defined, we should now be able to test OPA Gatekeeper to make sure it's enforcing our policy.

Now let's create a resource of kind Namespace without the "manager" label to test our constraint:

In the Rancher UI, click on the dropdown menu in the top left-hand corner and select 'Cluster Explorer'.
From the menu on the left, click 'Pods' under the 'Workload' section.
Click on the 'Create from YAML' button and paste in the following Pod definition, overwriting the existing example template:
apiVersion: v1
kind: Namespace
metadata:
 labels:
     owner: "true"
 name: test
Click on 'Create'.
After creating the Pod, you should have received an error about not finding the "manager" label.

