

# 1. Create S3 Bucket and versioning. 

aws s3api create-bucket --bucket cilistapp-kops-state \
--region ap-southeast-1 --create-bucket-configuration \
LocationConstraint=ap-southeast-1

LocationConstraint=ap-southeast-1
{
    "Location": "http://cilistapp-kops-state.s3.amazonaws.com/"
}

aws s3api put-bucket-versioning --bucket cilistapp-kops-state \
--region ap-southeast-1 --versioning-configuration Status=Enabled

# 2. Create Kubernetes cluster use kops

# A. Topology
# By default kOps will create clusters using public topology, where all nodes and the Kubernetes API 
# are exposed on public Internet. If topology is private, All masters/nodes will be launched in a private 
# subnet in the VPC

# If a subnet's traffic is routed to an Internet gateway, the subnet is known as a public subnet.
# If a subnet doesn't have a route to the Internet gateway, the subnet is known as a private subnet.
# Private topologies will have public access via the Kubernetes API and an (optional) SSH bastion instance.

# B. Networking
# The default networking of kOps, kubenet, is not recommended for production. Most importantly, it does not 
# support network policies, nor does it support internal networking. Hence, CNI will be used.

export KOPS_STATE_STORE="s3://cilistapp-kops-state"
export MASTER_SIZE="t2.medium"
export NODE_SIZE="t2.medium"
export ZONES="ap-southeast-1a,ap-southeast-1b,ap-southeast-1c"

kops create cluster --name=cilistapp.k8s.local \
--zones=$ZONES \
--master-zones=$ZONES \
--master-size=$MASTER_SIZE \
--node-size=$NODE_SIZE \
--node-count=3 \
--networking=calico \
--topology=private \
--ssh-public-key=.ssh/authorized_keys \
--state=$KOPS_STATE_STORE 

Cluster configuration has been created.

Suggestions:
 * list clusters with: kops get cluster
 * edit this cluster with: kops edit cluster cilistapp.k8s.local
 * edit your node instance group: kops edit ig --name=cilistapp.k8s.local nodes-ap-southeast-1a
 * edit your master instance group: kops edit ig --name=cilistapp.k8s.local master-ap-southeast-1a

Finally configure your cluster with: kops update cluster --name cilistapp.k8s.local --yes --admin

kops update cluster --name cilistapp.k8s.local --state s3://cilistapp-kops-state --yes --admin

Cluster is starting.  It should be ready in a few minutes.

Suggestions:
 * validate cluster: kops validate cluster --wait 10m
 * list nodes: kubectl get nodes --show-labels
 * to ssh to the bastion, you probably want to configure a bastionPublicName.
 * the ubuntu user is specific to Ubuntu. If not using Ubuntu please use the appropriate user based on your OS.
 * read about installing addons at: https://kops.sigs.k8s.io/addons.

kops get cluster --state s3://cilistapp-kops-state

# ssh to bastian host
% aws elb --output=table describe-load-balancers|grep DNSName.\*bastion|awk '{print $4}'
bastion-cilistapp-k8s-loc-rrckl5-1302333394.ap-southeast-1.elb.amazonaws.com

ssh -i awskilay-key.pem ubuntu@bastion-cilistapp-k8s-loc-rrckl5-1302333394.ap-southeast-1.elb.amazonaws.com

# ssh to private instance.

create id_rsa inside .ssh from local id_rsa.

$ vi id_rsa

-----BEGIN RSA PRIVATE KEY-----
MIIEowIBAAKCAQEAgtdSwrn/C/+z0bnpf9qvyDKvauDoypVK2GmLp0Tz55b3ybsu
yi7h18oAj6pGQwQIfehZHGncNd94hRIIZInDA5Mnb197wpTF3B5Xar63XNjvrukX
yetCQQsIyFuL2lMdaGU/8NzDMmgamDoK86sED9HvPdSnO+rccqGeFnbROVzZoEvf
/qNySRTm6ems1yUZQkrmGUTm2NkUwL9v4HyuwsQzhRWLV7/RcIpENDXP6FQbk3EC
GBttUZ/1tQmLUN+BLjwZ+FS1v36FKmMYdyQKAXg+1nH440ArLtgsmHT9O5iG4YUN
xtJi2/kBgfJXoHkEt+nSTdKGaWZJcjWRxAV3QwIDAQABAoIBAAm/p1/w4crwE2LV
+krXbW96L03EUjP96aS0QH6HCbFAs1ephbP0yEj+uQn7Qt7tZwCSlkkirhCphN5N
WKi9BvW2OiL3N05pLVDYReUjLqBRXZJntakKyVX1T4M2JvZuaOuFV71HhZe03/5l
nLlJDbVsC+pMdOVm+2PjHNdJpQ8j+5+C60sdVYPoZur4V9taIdEvb05So9kgSlbz
xx2HejSzxAGTFWMLF9CB0HLWcGf91UgBqy3eu+5AaQW+7373sqnBYf3o5lpxXjm4
0nwJesC/VDUMe65whBNedE2m2AQxySEDUDs6aCFe3AlgfpwWI3AniWQ1cg0ZOs7Z
F6QTxPECgYEAuyORnc8GZkHITaEFUpV8SYI3Y1eMupdXdR6z7fEw+3WRxRGvSTF7
zUkPeyo5RPZSxrD96iipcmo4tXr3CbMh+KIBn/gYCftdLHP84JIZa48Yg1o+GXbB
bVWGqcDtkVrLUNccqa6IBPTh9sFpZ2blUChBaowL63D4N4buvb3MDusCgYEAsvyE
Ac3bZt8XmruxZx31FWiq5MFNhx+vDZyWHtT/sgitOvMoqLCK2i5rX8jGAZCIAG70
vDZvhseCjQYemq5m5CqAowknvSdjhsp564Qonjmk1U/QOTwlsaO134bjjXtQUEO9
T0DIbzcYBEqL+tZLOVfFrunWomLyS5Wd1L/YkwkCgYB0I/n+Z3qAQfku/GzSOQXe
lRsM40vqjXxwqnJejJ6qoOer13LiyPwdhmc+OBE81GbA+x1Kkpu+719sefkRIwRF
Sz4Y6p74qvDDYuSg9ushzrgW5Q2/Pe2Djl25wott91xROn+Ga1PtR5FpU9W3n6tX
WPRoTKwlHYJe67YFOeKHqwKBgQCwxuY6Qe9ocv8FPEvC5LujIXVn6eOAibKDZxx9
5zGDzT4K8w49TeBWDXLPb6Tg9rbcdroRClKsc3BliJ3BeG72+2OBoxE0qSqLfn9c
NXNIkvZSGDo3zUgNYvvGgZtNqXVxUPYwyHMuJOP7mQUYAX7aa+47C4mJaOCV9nek
ILbuwQKBgF1ue0kPyA6xf/bzu6AqnL/FRQYrNKBWn0hPqiXu3NNIe5H7IWSpANFa
y6fx4yckpEc3aTmd55tMt6Yrr+5C/ObagtXoQ7hodQyfLUZKKtLTc49YeeRVreP0
PEzHDTlufl5KWvQDZvdbOLpDCm747VWRt4mBCLOCm2zHdTKvF2z8
-----END RSA PRIVATE KEY-----

$ chmod 400 id_rsa

# to add bastion host a public IP.
kops edit ig bastions --name cilistapp.k8s.local --state s3://cilistapp-kops-state
associatePublicIp: true

NAME			CLOUD	ZONES
cilistapp.k8s.local	aws	ap-southeast-1a,ap-southeast-1b,ap-southeast-1c

kops validate cluster --name cilistapp.k8s.local --state s3://cilistapp-kops-state --wait 10m 

INSTANCE GROUPS
NAME			ROLE	MACHINETYPE	MIN	MAX	SUBNETS
bastions		Bastion	t3.micro	1	1	ap-southeast-1a,ap-southeast-1b,ap-southeast-1c
master-ap-southeast-1a	Master	t2.medium	1	1	ap-southeast-1a
master-ap-southeast-1b	Master	t2.medium	1	1	ap-southeast-1b
master-ap-southeast-1c	Master	t2.medium	1	1	ap-southeast-1c
nodes-ap-southeast-1a	Node	t2.small	1	1	ap-southeast-1a
nodes-ap-southeast-1b	Node	t2.small	1	1	ap-southeast-1b
nodes-ap-southeast-1c	Node	t2.small	1	1	ap-southeast-1c

NODE STATUS
NAME			ROLE	READY
i-060ea09e114044f09	node	True
i-094ff4e1021ff76ff	node	True
i-09bf30851abed518b	master	True
i-09dc5380caebc44b1	node	True
i-0bf6a6ac215735523	master	True
i-0ea4b6190b5d5cd2a	master	True

Your cluster cilistapp.k8s.local is ready

% kubectl get nodes
NAME                  STATUS   ROLES           AGE   VERSION
i-060ea09e114044f09   Ready    node            25m   v1.24.4
i-094ff4e1021ff76ff   Ready    node            25m   v1.24.4
i-09bf30851abed518b   Ready    control-plane   26m   v1.24.4
i-09dc5380caebc44b1   Ready    node            25m   v1.24.4
i-0bf6a6ac215735523   Ready    control-plane   26m   v1.24.4
i-0ea4b6190b5d5cd2a   Ready    control-plane   26m   v1.24.4

% kubectl get cs   
NAME                 STATUS    MESSAGE                         ERROR
scheduler            Healthy   ok                              
controller-manager   Healthy   ok                              
etcd-1               Healthy   {"health":"true","reason":""}   
etcd-0               Healthy   {"health":"true","reason":""}   

# 3. Install Jenkins

# 4. Install Ansible + Docker

# 5. Create AWS ECR

# 6. install helm

curl https://baltocdn.com/helm/signing.asc | gpg --dearmor | sudo tee /usr/share/keyrings/helm.gpg > /dev/null
sudo apt-get install apt-transport-https --yes
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/helm.gpg] https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
sudo apt-get update
sudo apt-get install helm

helm upgrade --install ingress-nginx ingress-nginx \
  --repo https://kubernetes.github.io/ingress-nginx \
  --namespace ingress-nginx --create-namespace

% kubectl get pod -n ingress-nginx                    
NAME                                        READY   STATUS    RESTARTS   AGE
ingress-nginx-controller-6bf7bc7f94-4bfmm   1/1     Running   0          29s
irwankilay@irwans-iMac ~ % kubectl get all -n ingress-nginx 
NAME                                            READY   STATUS    RESTARTS   AGE
pod/ingress-nginx-controller-6bf7bc7f94-4bfmm   1/1     Running   0          44s

NAME                                         TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
service/ingress-nginx-controller             LoadBalancer   100.66.220.157   <pending>     80:30787/TCP,443:31820/TCP   44s
service/ingress-nginx-controller-admission   ClusterIP      100.67.187.203   <none>        443/TCP                      44s

NAME                                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/ingress-nginx-controller   1/1     1            1           44s

NAME                                                  DESIRED   CURRENT   READY   AGE
replicaset.apps/ingress-nginx-controller-6bf7bc7f94   1         1         1       44s

# 7.install cert manager

kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.9.1/cert-manager.yaml

% kubectl get pod -n cert-manager
NAME                                       READY   STATUS              RESTARTS   AGE
cert-manager-5dd59d9d9b-hl94g              0/1     ContainerCreating   0          7s
cert-manager-cainjector-8696fc9f89-jvnxl   0/1     ContainerCreating   0          7s
cert-manager-webhook-7d4b5b8c56-fwffg      0/1     ContainerCreating   0          7s

# 8.install cluster-issuer yaml.

% kubectl apply -f cluster-issuer.yaml 
clusterissuer.cert-manager.io/mern-prod created
clusterissuer.cert-manager.io/mern-staging created

% kubectl get certificate -n staging
NAME               READY   SECRET             AGE
mern-staging-tls   True    mern-staging-tls   2m6s

kubectl get certificate -n production
NAME            READY   SECRET          AGE
mern-prod-tls   True    mern-prod-tls   79s

# 9.install ingress rule for staging and production

% kubectl apply -f ingress-staging.yaml 
ingress.networking.k8s.io/mern-ingress created

% vi ingress-production.yaml
% kubectl apply -f ingress-production.yaml 


# 10. install metric server

% kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
serviceaccount/metrics-server created
clusterrole.rbac.authorization.k8s.io/system:aggregated-metrics-reader created
clusterrole.rbac.authorization.k8s.io/system:metrics-server created
rolebinding.rbac.authorization.k8s.io/metrics-server-auth-reader created
clusterrolebinding.rbac.authorization.k8s.io/metrics-server:system:auth-delegator created
clusterrolebinding.rbac.authorization.k8s.io/system:metrics-server created
service/metrics-server created
deployment.apps/metrics-server created
apiservice.apiregistration.k8s.io/v1beta1.metrics.k8s.io created
irwankilay@irwans-iMac ~ % kubectl apply -f ingress-production.yaml

% kubectl top nodes
NAME                  CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%   
i-060ea09e114044f09   56m          5%     1110Mi          59%       
i-094ff4e1021ff76ff   55m          5%     834Mi           44%       
i-09bf30851abed518b   192m         9%     1955Mi          51%       
i-09dc5380caebc44b1   55m          5%     916Mi           48%       
i-0bf6a6ac215735523   187m         9%     2053Mi          53%       
i-0ea4b6190b5d5cd2a   212m         10%    2093Mi          54% 

 % kubectl top pods -n staging 
NAME                             CPU(cores)   MEMORY(bytes)   
mern-backend-b766d87d-xntfk      0m           62Mi            
mern-frontend-7cdd5fb4c6-x82wf   1m           351Mi    

# 11. Create RDS

setup database people, username: admin, password: iwankilay 

# 12. Setup Route53
Setup staging.irwankilay.com and production.irwankilay.com -> load balancer.

# 13. Deploy source code

# 14. Install Instance group for monitoring

$ kops get instancegroup
Using cluster from kubectl context: cilistapp.k8s.local

NAME			ROLE	MACHINETYPE	MIN	MAX	ZONES
bastions		Bastion	t3.micro	1	1	ap-southeast-1a,ap-southeast-1b,ap-southeast-1c
master-ap-southeast-1a	Master	t2.medium	1	1	ap-southeast-1a
master-ap-southeast-1b	Master	t2.medium	1	1	ap-southeast-1b
master-ap-southeast-1c	Master	t2.medium	1	1	ap-southeast-1c
nodes-ap-southeast-1a	  Node	  t2.small	1	1	ap-southeast-1a
nodes-ap-southeast-1b	  Node	  t2.small	1	1	ap-southeast-1b
nodes-ap-southeast-1c	  Node	  t2.small	1	1	ap-southeast-1c

$ kops get instancegroup nodes-ap-southeast-1a -o yaml > kops-ig-monitoring.yaml
Using cluster from kubectl context: cilistapp.k8s.local

$ kops create -f kops-ig-monitoring.yaml --> refer to ikhsan yaml
kops update cluster --name cilistapp.k8s.local --state s3://cilistapp-kops-state --yes
$ kops rolling-update cluster
$ kops validate cluster

# 15. Install Jenkins 


wget -q -O - https://pkg.jenkins.io/debian/jenkins.io.key | sudo apt-key add -

sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > \
/etc/apt/sources.list.d/jenkins.list'

sudo apt-get update
sudo apt-get install openjdk-8-jdk
sudo apt-get install jenkins

sudo systemctl status jenkins

sudo hostnamectl set-hostname jenkins-svr

username : admin
password : iwankilay

# 16. Install Ansible-Docker

$ sudo apt-get update
$ sudo apt-get install python ansible

$ curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
$ unzip awscliv2.zip
$ sudo ./aws/install

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

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

kops export kubecfg --admin

grafana user     : admin
grafana password : prom-operator
