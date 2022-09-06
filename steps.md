

# 1. Create S3 Bucket and versioning. 

$ aws s3api create-bucket --bucket cilistapp-kops-state \
--region ap-southeast-1 --create-bucket-configuration \
LocationConstraint=ap-southeast-1

$ aws s3api put-bucket-versioning --bucket cilistapp-kops-state \
--region ap-southeast-1 --versioning-configuration Status=Enabled

# 2. Create RDS

setup database people, username: admin, password: iwankilay 

# 3. Create Kubernetes cluster use kops

$ export KOPS_STATE_STORE="s3://cilistapp-kops-state"
$ export MASTER_SIZE="t2.medium"
$ export NODE_SIZE="t2.medium"
$ export ZONES="ap-southeast-1a,ap-southeast-1b,ap-southeast-1c"

$ kops create cluster --name=cilistapp.k8s.local \
--zones=$ZONES \
--master-zones=$ZONES \
--master-size=$MASTER_SIZE \
--node-size=$NODE_SIZE \
--node-count=3 \
--networking=calico \
--topology=private \
--ssh-public-key=.ssh/authorized_keys \
--state=$KOPS_STATE_STORE 

$ kops update cluster --name cilistapp.k8s.local --state s3://cilistapp-kops-state --yes --admin

$ kops validate cluster --name cilistapp.k8s.local --state s3://cilistapp-kops-state --wait 10m 

$ kops get cluster --state s3://cilistapp-kops-state

# 4. Install Jenkins+ Docker + kubectl

------- Jenkins ---------------

create EC2 type medium.

$ sudo hostnamectl set-hostname jenkins-svr

$ wget -q -O - https://pkg.jenkins.io/debian/jenkins.io.key | sudo apt-key add -

$ sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > \
/etc/apt/sources.list.d/jenkins.list'

$ sudo apt-get update
$ sudo apt-get install openjdk-8-jdk
$ sudo apt-get install jenkins

sudo systemctl status jenkins

------- Docker ---------------

$ sudo apt-get update
$ sudo apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release

$ sudo mkdir -p /etc/apt/keyrings
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
$ echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

$ sudo apt-get update
$ sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin

--------Kubectl ----------------

$ curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

$ sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

# 5. install helm

curl https://baltocdn.com/helm/signing.asc | gpg --dearmor | sudo tee /usr/share/keyrings/helm.gpg > /dev/null
sudo apt-get install apt-transport-https --yes
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/helm.gpg] https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
sudo apt-get update
sudo apt-get install helm

# 6. install ingress

$ helm upgrade --install ingress-nginx ingress-nginx \
  --repo https://kubernetes.github.io/ingress-nginx \
  --namespace ingress-nginx --create-namespace

# 7.install cert manager

kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.9.1/cert-manager.yaml

# 8.install cluster-issuer yaml.

% kubectl apply -f cluster-issuer.yaml 

% kubectl get certificate -n staging

# 9.install ingress rule for staging and production

% kubectl apply -f ingress-staging.yaml 

% kubectl apply -f ingress-production.yaml


# 10. install metric server

% kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

# 11. Setup Route53
Setup staging.irwankilay.com and production.irwankilay.com -> load balancer.

# 12. Install Instance group for monitoring

$ kops get instancegroup

$ kops get instancegroup nodes-ap-southeast-1a -o yaml > kops-ig-monitoring.yaml

$ kops create -f kops-ig-monitoring.yaml 

$ kops rolling-update cluster

$ kops validate cluster

# 12. Install promotheus + grafana

grafana user     : admin
grafana password : prom-operator

# 13. Install Loki
