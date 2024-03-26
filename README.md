#K8s

Kubernetes project:

1. Frontend Dockerfile:

# Use an official Node.js runtime as a base image
FROM node:14-alpine

# Set the working directory in the container
WORKDIR /app

# Copy package.json and package-lock.json to the working directory
COPY package*.json ./

# Install dependencies
RUN npm install

# Copy the rest of the application code to the working directory
COPY . .

# Build the frontend application
RUN npm run build

# Expose port 3000 to the outside world
EXPOSE 3000

# Command to run the frontend application
CMD ["npm", "start"]


Backend Dockerfile 

# Use an official Python runtime as a base image
FROM python:3.9-slim

# Set the working directory in the container
WORKDIR /app

# Copy requirements.txt to the working directory
COPY requirements.txt .

# Install Python dependencies
RUN pip install --no-cache-dir -r requirements.txt

# Copy the rest of the application code to the working directory
COPY . .

# Expose port 5000 to the outside world
EXPOSE 5000

# Command to run the backend application
CMD ["python", "app.py"]


Build Docker images:

cd /path/to/frontend
docker build -t your-frontend-image-name .


cd /path/to/backend
docker build -t your-backend-image-name .

docker images

 2. Design Kubernetes deployment:
 
 Frontend Deployment Manifest (frontend-deployment.yaml):
 
 apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend-deployment
spec:
  replicas: 3  # Adjust the number of replicas as needed
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
      - name: frontend
        image: your-frontend-image-name  # Replace with the name of your frontend Docker image
        ports:
        - containerPort: 3000  # Expose the same port as specified in the Dockerfile
        resources:
          limits:
            cpu: "0.5"  # Adjust CPU limit as needed
            memory: "256Mi"  # Adjust memory limit as needed
          requests:
            cpu: "0.1"  # Adjust CPU request as needed
            memory: "128Mi"  # Adjust memory request as needed
        env:
          # Define any required environment variables for the frontend application
          - name: API_URL
            value: "http://backend-service:5000"  # Example environment variable for backend API URL
        # Define logging and monitoring configurations for the frontend container
        lifecycle:
          preStop:
            exec:
              command: ["/bin/sh", "-c", "your-log-cleanup-command"]  # Example log cleanup command



Backend Deployment Manifest (backend-deployment.yaml):

apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend-deployment
spec:
  replicas: 3  # Adjust the number of replicas as needed
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
    spec:
      containers:
      - name: backend
        image: your-backend-image-name  # Replace with the name of your backend Docker image
        ports:
        - containerPort: 5000  # Expose the same port as specified in the Dockerfile
        resources:
          limits:
            cpu: "0.5"  # Adjust CPU limit as needed
            memory: "512Mi"  # Adjust memory limit as needed
          requests:
            cpu: "0.1"  # Adjust CPU request as needed
            memory: "256Mi"  # Adjust memory request as needed
        env:
          # Define any required environment variables for the backend application
          - name: DATABASE_URL
            value: "your-database-url"  # Example environment variable for database connection
        # Define logging and monitoring configurations for the backend container
        lifecycle:
          preStop:
            exec:
              command: ["/bin/sh", "-c", "your-log-cleanup-command"]  # Example log cleanup command



The frontend deployment manifest specifies the desired number of replicas, selects the frontend pods based on the app: frontend label, and specifies the frontend Docker image to use. It also exposes port 3000, which is the same port specified in the frontend Dockerfile. Additionally, it defines any required environment variables, such as the backend API URL.
The backend deployment manifest follows a similar structure but applies to the backend component. It specifies the desired number of replicas, selects the backend pods based on the app: backend label, and specifies the backend Docker image to use. It exposes port 5000, as specified in the backend Dockerfile.
Make sure to replace your-frontend-image-name and your-backend-image-name with the names of your frontend and backend Docker images, respectively. Additionally, customize the environment variables and other settings as per your application requirements.


Resource limits and requests are defined for both frontend and backend containers to ensure efficient resource utilization.
Required environment variables are defined for each component according to the specified requirements.
Logging and monitoring configurations are added using the container lifecycle preStop hook. Replace your-log-cleanup-command with the appropriate command for log cleanup.
Make sure to replace your-frontend-image-name, your-backend-image-name, and other placeholder values with the actual values for your application. Additionally, customize the resource limits, environment variables, and logging configurations as per your application requirements.

3. Implement Kubernetes service:

frontend-service.yaml

apiVersion: v1
kind: Service
metadata:
  name: frontend-service
spec:
  selector:
    app: frontend
  ports:
    - protocol: TCP
      port: 80
      targetPort: 3000
  type: ClusterIP


apiVersion: Specifies the version of the Kubernetes API used by the manifest.
kind: Defines the type of Kubernetes resource, which in this case is a Service.
metadata: Contains metadata for the service, including its name.
spec: Specifies the desired state for the service.
selector: Defines a set of labels used to identify the pods to which the service should forward traffic. In this case, it selects pods with the label app: frontend.
ports: Specifies the ports that the service should listen on and forward traffic to.
protocol: Specifies the protocol used by the ports, which is TCP in this case.
port: Specifies the port on which the service should listen for incoming traffic.
targetPort: Specifies the port on the pods to which the traffic should be forwarded. This should match the port exposed by the frontend container (3000 in this case).
type: Specifies the type of service. ClusterIP exposes the service on an internal IP within the cluster. This ensures that the service is accessible only from within the cluster.

kubectl apply -f frontend-service.yaml


After applying the service manifest, the frontend service will be created, and it will forward traffic to pods labeled with app: frontend. This service will be accessible within the cluster using the ClusterIP assigned to it.

 4. Configure application scaling:

backend-hpa.yaml

apiVersion: autoscaling/v2beta2
kind: HorizontalPodAutoscaler
metadata:
  name: backend-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: backend-deployment  # Name of your backend deployment
  minReplicas: 1  # Minimum number of replicas
  maxReplicas: 10  # Maximum number of replicas
  metrics:
  - type: Resource
    resource:
      name: cpu
      targetAverageUtilization: 50  # Target CPU utilization percentage for scaling


Explanation of the HPA manifest:

apiVersion: Specifies the API version of the HorizontalPodAutoscaler resource.
kind: Defines the type of Kubernetes resource, which is HorizontalPodAutoscaler.
metadata: Contains metadata for the HPA, including its name.
spec: Specifies the desired state for the HPA.
scaleTargetRef: Specifies the deployment to which the HPA should scale the number of replicas.
apiVersion: Specifies the API version of the target deployment.
kind: Specifies the kind of the target deployment, which is Deployment in this case.
name: Specifies the name of the target deployment.
minReplicas: Defines the minimum number of replicas for the backend deployment.
maxReplicas: Defines the maximum number of replicas for the backend deployment.
metrics: Specifies the metrics used for autoscaling.
type: Specifies the type of metric, which is Resource in this case.
resource: Specifies the resource metric source.
name: Specifies the name of the resource, which is CPU in this case.
targetAverageUtilization: Defines the target average CPU utilization percentage for scaling.
To apply this HPA manifest, use the kubectl apply command:

kubectl apply -f backend-hpa.yaml


After applying the HPA manifest, Kubernetes will automatically adjust the number of backend replicas based on CPU utilization, ensuring that it stays within the defined minimum and maximum replica limits. Adjust the targetAverageUtilization, minReplicas, and maxReplicas values according to your application's requirements and resource availability.


====================================================================

Kubernetes
---------
Architecture
set up
Pods spectifications 
various application installations on k8s ( like nginx, tomcat, mogodb,sql, wordpress etc ...)

Kubernetes:
Kubernetes is a Production-Grade Container Orchestration where it manages automated container deployment, scaling, and management.

Architecture:
-------------
Pic refer


Kubernetes master & node configuration set up:

Prerequisite: minimum test server requirement for practice prupose 
=============
3 - Ubuntu Servers
1 - Manager  (4GB RAM , 2 Core) t2.medium
2 - Workers  (1 GB, 1 Core)     t2.micro

Kubernetes both Master& Node server installation step by step  needed to follow:
---------------
1-Switch to root user
   $sudo su -
2-Disable swap & add kernel settings
   swapoff -a
   sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab  ( permanently disable on fstab )


3-Add  kernel settings & Enable IP tables(CNI Prerequisites)
---
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF


modprobe overlay
modprobe br_netfilter



cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF


sysctl --system
----
4- Install containerd run time & configure the containerd
apt-get update -y 
apt-get install ca-certificates curl gnupg lsb-release -y
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
apt-get update -y
apt-get install containerd.io -y
containerd config default > /etc/containerd/config.toml
sed -i 's/SystemdCgroup \= false/SystemdCgroup \= true/g' /etc/containerd/config.toml

5- Restart and enable containerd service
systemctl restart containerd
systemctl enable containerd

check the containerd status: 
systemctl status containerd

6- Installing kubeadm, kubelet and kubectl 
apt-get update
apt-get install -y apt-transport-https ca-certificates curl
curl -fsSL https://dl.k8s.io/apt/doc/apt-key.gpg | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-archive-keyring.gpg
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list

Update apt package index, install kubelet, kubeadm and kubectl, and pin their version:

apt-get update
apt-get install -y kubelet kubeadm kubectl

apt-mark hold will prevent the package from being automatically upgraded or removed.

apt-mark hold kubelet kubeadm kubectl

Enable and start kubelet service:

systemctl daemon-reload 
systemctl start kubelet 
systemctl enable kubelet.service

--------------------------------------------------------------------

Master server only steps to  Start:

sudo su -
kubeadm init
kubeadm init --control-plane-endpoint "PUBLIC_IP:PORT"  ( to check the public ip address on AWS & port is 6443)
IF Error
sudo kubeadm init --cri-socket /run/containerd/containerd.sock

Configure kubectl  exit as root user & exeucte as normal ubuntu user
exit

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

To verify, if kubectl is working or not, run the following command.
kubectl get pods -o wide -n kube-system

Note: Install any one network addon don't install both. Install either weave net or calico.
kubectl apply -f https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s.yaml

kubectl get nodes

kubectl get pods 
kubectl get pods --all-namespaces


# Get token:

kubeadm token create --print-join-command

ubuntu@ip-172-31-8-202:~/.kube$ kubeadm token create --print-join-command

kubeadm join 172.31.47.176:6443 --token u2o9np.31gd3rl0dw5vht41 --discovery-token-ca-cert-hash sha256:3e97777ed25572bdde286345f3663009c726f634302d78e5d8e02b75a176293f


Need to run below commaand on worker node to join with master server:
kubeadm join 172.31.47.176:6443 --token u2o9np.31gd3rl0dw5vht41 --discovery-token-ca-cert-hash sha256:3e97777ed25572bdde286345f3663009c726f634302d78e5d8e02b75a176293f


To test the master & work node connection made or not:

kubectl get nodes

kubectl get pods 
kubectl get pods --all-namespaces


Various commands to use case on Kubernetes:
---------------------------------------------
To check the nodes : 
kubectl get nodes

To check the pods 
kubectl get pods -n <namespacename> ( to check particular namespace )

To check all pods with all namespaces:
kubectl get pods --all-namespaces

To create a namespace 
kubectl create namespace test-ns
k get ns
k config set-context --current --namespace=test-ns 

To show pod details:
kubectl describe pod nginx-deployment-86dcfdf4c6-kc7vf




Creating Deployment POD for nginx
---------------------------------

apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: nginxapp
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
  - name: name-of-service-port
    protocol: TCP
    port: 80
    targetPort: 80


Error :
-------
  Type     Reason                  Age                  From               Message
  ----     ------                  ----                 ----               -------
  Normal   Scheduled               9m44s                default-scheduler  Successfully assigned default/nginx-deployment-86dcfdf4c6-kc7vf to ip-172-31-45-58
  Warning  FailedCreatePodSandBox  90s (x2 over 5m43s)  kubelet            Failed to create pod sandbox: rpc error: code = DeadlineExceeded desc = context deadline exceeded

Solution:
 need to check on resource matrix like cpu & ram utilization for worker node. And increase the resource 


Install Helm package in Ubuntu:

curl https://baltocdn.com/helm/signing.asc | gpg --dearmor | sudo tee /usr/share/keyrings/helm.gpg > /dev/null
sudo apt-get install apt-transport-https --yes
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/helm.gpg] https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
sudo apt-get update
sudo apt-get install helm


Instal Prometheus on K8S:

https://medium.com/@vinoji2005/install-prometheus-on-kubernetes-tutorial-and-example-6b3c800e7e1c

http://<EXTERNAL_IP_ADDRESS>:Portno 

======================================================================================





==========================================================================

Kubernetes CGI Int topics:

K8s, kubespray, kubeadm, Sidecar containers, Init containers, Static pods and purpose, Stateful Set,
Daemon set, Services types, Pod issue, restrict pod communication, Pod & PVC, Deployment rolling strategy
Diff branching strategies, git stash, git rebase
Helm charts

==========================================================================


Managed kubernetes ( EKS- Amazon Elastic Kubernetes Service ) 
--------------------------------------------------------------
Agenda:
-----
GUI process to create the EKS cluster 
Setup & configuration 
various Pod creation 
Various application deploy to EKS


Configutration Steps:
1- Create VPC and minimum 2 subnet ( private & public )
2- Create IAM role for EKS controller
3- Create EKS Cluster
4- Create IAM role for EKS worker node


1)VPC creation with other configuration:
10.0.0.0/20 - VPC CIDR range 

Created subnet ( 2 private & 2 public with below range)(two different availability zone to create subnet)

10.0.0.0/22	10.0.0.0 - 10.0.3.255	10.0.0.1 - 10.0.3.254	1022	Divide			
10.0.4.0/22	10.0.4.0 - 10.0.7.255	10.0.4.1 - 10.0.7.254	1022	Divide	
10.0.8.0/22	10.0.8.0 - 10.0.11.255	10.0.8.1 - 10.0.11.254	1022	Divide		
10.0.12.0/22    10.0.12.0 - 10.0.15.255	10.0.12.1 - 10.0.15.254	1022	Divide	

Created IGW and attach to VPC

Created NAT and associate with Elastic IP

Created 2 Route table 
  privateRT and associate with private subnet And Route mentioned with 0.0.0.0/0 with NAT-id
  PublicRT and associate with public subnet And Route mentioned with 0.0.0.0/0  with IWG-id


2) Create IAM role for EKS cluster

3) Create EKS cluster and attach EKSrole

4) Create IAM role for EKS worker attach with below policy
      AmazonEKSWorkerNodePolicy
      AmazonEKS_CNI_Policy
      AmazonEC2ContainerRegistryReadOnly
      AmazonEBSCSIDriverPolicy 
5) Create Nodegroup   

   
To connection process for  EKS_cluster 

Make a client_server 
Then install Kubectl 
   curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
   sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
   chmod +x kubectl
   mkdir -p ~/.local/bin
   mv ./kubectl ~/.local/bin/kubectl
then install aws cli 
   curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
   unzip awscliv2.zip
   sudo ./aws/install

aws cli configure on this server 
  $aws configure 
      access_key:AKIAZ5QWTGKYFZT2UWVY
      secret_key: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
  $aws sts get-caller-identity ( to check the aws account has connected or not )

  $aws eks update-kubeconfig --name Eks_cluster_1  --region ap-south-1    

  $kubectl apply -k "github.com/kubernetes-sigs/aws-ebs-csi-driver/deploy/kubernetes/overlays/stable/?ref=release-1.12"


Helm package install : 

$ curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
$ chmod 700 get_helm.sh
$ ./get_helm.sh
  helm 


helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm install  prometheus prometheus-community/prometheus

Referance link for helm packackage
https://helm.sh/docs/intro/cheatsheet/


Get the Prometheus server URL by running these commands in the same shell:
  export POD_NAME=$(kubectl get pods --namespace default -l "app.kubernetes.io/name=prometheus,app.kubernetes.io/instance=prometheous" -o jsonpath="{.items[0].metadata.name}")
  kubectl --namespace default port-forward $POD_NAME 9090


The Prometheus alertmanager can be accessed via port 9093 on the following DNS name from within your cluster:
prometheous-alertmanager.default.svc.cluster.local


Get the Alertmanager URL by running these commands in the same shell:
  export POD_NAME=$(kubectl get pods --namespace default -l "app.kubernetes.io/name=alertmanager,app.kubernetes.io/instance=prometheous" -o jsonpath="{.items[0].metadata.name}")
  kubectl --namespace default port-forward $POD_NAME 9093
#################################################################################
######   WARNING: Pod Security Policy has been disabled by default since    #####
######            it deprecated after k8s 1.25+. use                        #####
######            (index .Values "prometheus-node-exporter" "rbac"          #####
###### .          "pspEnabled") with (index .Values                         #####
######            "prometheus-node-exporter" "rbac" "pspAnnotations")       #####
######            in case you still need it.                                #####
#################################################################################


The Prometheus PushGateway can be accessed via port 9091 on the following DNS name from within your cluster:
prometheous-prometheus-pushgateway.default.svc.cluster.local


Get the PushGateway URL by running these commands in the same shell:
  export POD_NAME=$(kubectl get pods --namespace default -l "app=prometheus-pushgateway,component=pushgateway" -o jsonpath="{.items[0].metadata.name}")
  kubectl --namespace default port-forward $POD_NAME 9091

For more information on running Prometheus, visit:
https://prometheus.io/



 my-release-g-grafana-654dff6bb5-mbqw2                 1/1     Running   0          4m29s   10.0.1.128   ip-10-0-3-130.ap-south-1.compute.internal   <none>           <none>
prometheous-alertmanager-0                            0/1     Pending   0          18m     <none>       <none>                                      <none>           <none>
prometheous-kube-state-metrics-7585c4484f-nm5cl       1/1     Running   0          18m     10.0.4.85    ip-10-0-5-207.ap-south-1.compute.internal   <none>           <none>
prometheous-prometheus-node-exporter-tkzzh            1/1     Running   0          18m     10.0.3.130   ip-10-0-3-130.ap-south-1.compute.internal   <none>           <none>
prometheous-prometheus-node-exporter-z48qx            1/1     Running   0          18m     10.0.5.207   ip-10-0-5-207.ap-south-1.compute.internal   <none>           <none>
prometheous-prometheus-pushgateway-54ff5fd47b-zsblk   1/1     Running   0          18m     10.0.6.225   ip-10-0-5-207.ap-south-1.compute.internal   <none>           <none>
prometheous-prometheus-server-68847c795-d7m4n         0/2     Pending   0          18m     <none>       <none>                                      <none>           <none>

ubuntu@ip-172-31-34-153:~$ ls
aws  awscliv2.zip  get_helm.sh  kubectl.sha256
ubuntu@ip-172-31-34-153:~$ kubectl get pods
NAME                                                  READY   STATUS    RESTARTS   AGE
my-release-g-grafana-654dff6bb5-mbqw2                 1/1     Running   0          10m
prometheous-alertmanager-0                            0/1     Pending   0          24m
prometheous-kube-state-metrics-7585c4484f-nm5cl       1/1     Running   0          24m
prometheous-prometheus-node-exporter-tkzzh            1/1     Running   0          24m
prometheous-prometheus-node-exporter-z48qx            1/1     Running   0          24m
prometheous-prometheus-pushgateway-54ff5fd47b-zsblk   1/1     Running   0          24m
prometheous-prometheus-server-68847c795-d7m4n         0/2     Pending   0          24m
ubuntu@ip-172-31-34-153:~$


ubuntu@ip-172-31-34-153:~$ kubectl get all --all-namespaces
NAMESPACE     NAME                                                      READY   STATUS    RESTARTS   AGE
default       pod/my-release-g-grafana-654dff6bb5-mbqw2                 1/1     Running   0          15m
default       pod/prometheous-alertmanager-0                            0/1     Pending   0          29m
default       pod/prometheous-kube-state-metrics-7585c4484f-nm5cl       1/1     Running   0          29m
default       pod/prometheous-prometheus-node-exporter-tkzzh            1/1     Running   0          29m
default       pod/prometheous-prometheus-node-exporter-z48qx            1/1     Running   0          29m
default       pod/prometheous-prometheus-pushgateway-54ff5fd47b-zsblk   1/1     Running   0          29m
default       pod/prometheous-prometheus-server-68847c795-d7m4n         0/2     Pending   0          29m
kube-system   pod/aws-node-bk6fg                                        2/2     Running   0          125m
kube-system   pod/aws-node-qj9mg                                        2/2     Running   0          125m
kube-system   pod/coredns-5d9d9bddbd-b69z9                              1/1     Running   0          126m
kube-system   pod/coredns-5d9d9bddbd-pq4mt                              1/1     Running   0          126m
kube-system   pod/eks-pod-identity-agent-8jqg8                          1/1     Running   0          125m
kube-system   pod/eks-pod-identity-agent-g8wtd                          1/1     Running   0          125m
kube-system   pod/kube-proxy-9m4x9                                      1/1     Running   0          125m
kube-system   pod/kube-proxy-xnf2z                                      1/1     Running   0          125m

NAMESPACE     NAME                                           TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)         AGE
default       service/kubernetes                             ClusterIP   172.20.0.1       <none>        443/TCP         137m
default       service/my-release-g-grafana                   ClusterIP   172.20.91.139    <none>        80/TCP          15m
default       service/prometheous-alertmanager               ClusterIP   172.20.127.71    <none>        9093/TCP        29m
default       service/prometheous-alertmanager-headless      ClusterIP   None             <none>        9093/TCP        29m
default       service/prometheous-kube-state-metrics         ClusterIP   172.20.66.85     <none>        8080/TCP        29m
default       service/prometheous-prometheus-node-exporter   ClusterIP   172.20.132.57    <none>        9100/TCP        29m
default       service/prometheous-prometheus-pushgateway     ClusterIP   172.20.184.112   <none>        9091/TCP        29m
default       service/prometheous-prometheus-server          ClusterIP   172.20.116.90    <none>        80/TCP          29m
kube-system   service/kube-dns                               ClusterIP   172.20.0.10      <none>        53/UDP,53/TCP   135m

NAMESPACE     NAME                                                  DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
default       daemonset.apps/prometheous-prometheus-node-exporter   2         2         2       2            2           kubernetes.io/os=linux   29m
kube-system   daemonset.apps/aws-node                               2         2         2       2            2           <none>                   135m
kube-system   daemonset.apps/eks-pod-identity-agent                 2         2         2       2            2           <none>                   126m
kube-system   daemonset.apps/kube-proxy                             2         2         2       2            2           <none>                   135m

NAMESPACE     NAME                                                 READY   UP-TO-DATE   AVAILABLE   AGE
default       deployment.apps/my-release-g-grafana                 1/1     1            1           15m
default       deployment.apps/prometheous-kube-state-metrics       1/1     1            1           29m
default       deployment.apps/prometheous-prometheus-pushgateway   1/1     1            1           29m
default       deployment.apps/prometheous-prometheus-server        0/1     1            0           29m
kube-system   deployment.apps/coredns                              2/2     2            2           135m

NAMESPACE     NAME                                                            DESIRED   CURRENT   READY   AGE
default       replicaset.apps/my-release-g-grafana-654dff6bb5                 1         1         1       15m
default       replicaset.apps/prometheous-kube-state-metrics-7585c4484f       1         1         1       29m
default       replicaset.apps/prometheous-prometheus-pushgateway-54ff5fd47b   1         1         1       29m
default       replicaset.apps/prometheous-prometheus-server-68847c795         1         1         0       29m
kube-system   replicaset.apps/coredns-56b8d964f7                              0         0         0       135m
kube-system   replicaset.apps/coredns-5d9d9bddbd                              2         2         2       126m

NAMESPACE   NAME                                        READY   AGE
default     statefulset.apps/prometheous-alertmanager   0/1     29m


Error:
Warning  FailedScheduling  85s (x3 over 21m)  default-scheduler  running PreBind plugin "VolumeBinding": binding volumes: timed out waiting for the condition

solution:
Installed aws-ebs-csi-driver  to solve this error.

kubectl apply -k "github.com/kubernetes-sigs/aws-ebs-csi-driver/deploy/kubernetes/overlays/stable/?ref=release-1.12"

Mongodb installation on EKS with the help of helm :

ubuntu@ip-172-31-34-153:~$ helm install my-release oci://registry-1.docker.io/bitnamicharts/mongodb
Pulled: registry-1.docker.io/bitnamicharts/mongodb:14.5.1
Digest: sha256:2f7ffc1e0b8c69fc572c092fe856946ac98d6e4733e20902571969e797399b74
NAME: my-release
LAST DEPLOYED: Wed Jan 17 09:52:03 2024
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
CHART NAME: mongodb
CHART VERSION: 14.5.1
APP VERSION: 7.0.5

** Please be patient while the chart is being deployed **

MongoDB&reg; can be accessed on the following DNS name(s) and ports from within your cluster:

    my-release-mongodb.default.svc.cluster.local

To get the root password run:

    export MONGODB_ROOT_PASSWORD=$(kubectl get secret --namespace default my-release-mongodb -o jsonpath="{.data.mongodb-root-password}" | base64 -d)

To connect to your database, create a MongoDB&reg; client container:

    kubectl run --namespace default my-release-mongodb-client --rm --tty -i --restart='Never' --env="MONGODB_ROOT_PASSWORD=$MONGODB_ROOT_PASSWORD" --image docker.io/bitnami/mongodb:7.0.5-debian-11-r0 --command -- bash

Then, run the following command:
    mongosh admin --host "my-release-mongodb" --authenticationDatabase admin -u $MONGODB_ROOT_USER -p $MONGODB_ROOT_PASSWORD

To connect to your database from outside the cluster execute the following commands:

    kubectl port-forward --namespace default svc/my-release-mongodb 27017:27017 &
    mongosh --host 127.0.0.1 --authenticationDatabase admin -p $MONGODB_ROOT_PASSWORD

===========================================================

Task on EKS: EKS Install and app deploy with Ingress

Github : https://github.com/iam-veeramalla/aws-devops-zero-to-hero/tree/main/day-22

Utube: https://www.youtube.com/watch?v=RRCrY12VY_s&t=25s

1. Creating Ec2 linux machine for Client server & created IAM User Eks-iam.

Access key: 

Secret key: 

Install Kubectl eksctl & AWS CLI:-
   curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
   sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
   chmod +x kubectl
   mkdir -p ~/.local/bin
   mv ./kubectl ~/.local/bin/kubectl
   kubectl
Install eksctl on Linux using curl:

	curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
	sudo mv /tmp/eksctl /usr/local/bin
	eksctl

aws cli 
   curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
   unzip awscliv2.zip
   sudo ./aws/install
   aws configure
   
2. Install EKS:

	eksctl create cluster --name demo-cluster --region us-east-1 --fargate
     
	To get Kubectl command line:
	
	aws eks update-kubeconfig --name demo-cluster --region us-east-1  
	
3. Creating Fargate profile	for 2048 app:

	eksctl create fargateprofile \
    --cluster demo-cluster \
    --region us-east-1 \
    --name alb-sample-app \
    --namespace game-2048
	
  Deploy the deployment, service and Ingress:
  
  kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/examples/2048/2048_full.yaml
  
  kubectl get pods -n game-2048 
  kubectl get svc -n game-2048 
  
  kubectl get ingress -n game-2048 


4. commands to configure IAM OIDC provider:

	eksctl utils associate-iam-oidc-provider --cluster demo-cluster --approve
	
5.setup alb add on:

   Download IAM policy:
   
   curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/install/iam_policy.json
   
   Create IAM Policy:
   
   aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://iam_policy.json
	
   Create IAM Role:
   
   eksctl create iamserviceaccount \
  --cluster=demo-cluster \
  --namespace=kube-system \
  --name=aws-load-balancer-controller \
  --role-name AmazonEKSLoadBalancerControllerRole \
  --attach-policy-arn=arn:aws:iam::681873453744:policy/AWSLoadBalancerControllerIAMPolicy \
  --approve
  
6.Deploy ALB controller:

  Helm package install : 

$ curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
$ chmod 700 get_helm.sh
$ ./get_helm.sh
  helm 

	Add helm repo:
	
	helm repo add eks https://aws.github.io/eks-charts
	
	Update the repo:
	
	helm repo update eks
	
	Installation:
	
	helm install aws-load-balancer-controller eks/aws-load-balancer-controller \            
  -n kube-system \
  --set clusterName=demo-cluster \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller \
  --set region=us-east-1 \
  --set vpcId=vpc-0604f6d1a29ebf916
  
  Verify that the deployments are running:
  
  kubectl get deployment -n kube-system aws-load-balancer-controller
  
  kubectl get ingress -n game-2048

  Ingress controller -> Ingress resource -> Load Balancer 

  Hit the DNS name from Load balancer in the browser:

  http://k8s-game2048-ingress2-bcac0b5b37-929505547.us-east-1.elb.amazonaws.com/
  
7. Deleting EKS Cluster:

  eksctl delete cluster --name demo-cluster --region us-east-1
  
=================================================================================================

K8s: COntainer Orchestration tool 

Architecture:

Master node -> Kube-API server, etcd server, Control manager, Scheduler

Worker node -> Kubelet, Pods, Containers, Kubeproxy.

Killercoda.com -> CKS

kubectl get pods -n kube-system

kubectl get nodes

kubectl describe nodes node01 

Master node + Worker node =Cluster
Kube-API server -> communication between master node components and worker node.
ETCD -> Inbuilt database, keep track of every cluster transaction that occurs.
Kube controller manager -> Monitor k8s objects and remediate the situation.
Kube Scheduler -> find best nodes for each containers.

Kubelet -> Operates pods from API server instructions
Kubeproxy -> maintain network between nodes and enables between Pods and external APIs

cd /etc/kubernetes/
cd manifests/
ls  -> to check all config yaml files 

vi pod.yaml -> copy from k8s docs

apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.14.2
    ports:
    - containerPort: 80

kubectl create -f pod.yaml
pod/nginx created
kubectl get pods
kubectl get pods -o wide 
kubectl describe pods

kubectl delete pods nginx 
kubectl get pods


replicaset -> No of pods
Deployment -> replicaset & no of pod details

k=kubectl 
k get rs
vi replicaset.yaml

apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: frontend
spec:
  replicas: 2
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      labels:
        tier: frontend
    spec:
      containers:
      - name: nginx
        image: nginx 



k create -f replicaset.yaml
k get rs
k describe rs frontend 
 

k get deploy
kubectl create deployment --image nginx:1.21 --port 80 --replicas=2 dep1
k get deploy 

k describe deploy <dep1> 

Namespace: Namespaces are a way to organize clusters into virtual sub-clusters (eg: State in a country)
k get namespace or k get ns
k create namespace dev 
k config set-context --current --namespace=dev 
k config view 
k config view | grep namespace 
k create -f pod.yaml --ns dev 

k get pods -A -> display all pods in all namespace 
k get all 

Scheduler:
vi pod.yaml 

apiVersion: v1
kind: Pod
metadata:
  name: my-pod
  labels:
   app: gw
   type: front-end
spec:
  containers:
  - name: c1
    image: nginx
    
k create -f pod.yaml

k get pods --selector app=gw 
k get pods

Services:
Cluster IP -> 

k config set-context --current --namespace=default
k get service 
k get svc

Diff Cluster IP needs to be communicated using Services

vi svc.yaml

apiVersion: v1
kind: Service 
metadata:
  name: my-service  
spec:
  ports:
   - targetPort: 80
     port: 80  

k create -f svc.yaml
k get svc

k get pods

k config set-context --current --namespace=dev
k get pods
vi svc.yaml -> change service name

apiVersion: v1
kind: Service 
metadata:
  name: your-service  
spec:
  ports:
    - targetPort: 80
	  port: 80	 

k create -f svc.yaml
k get svc
k edit svc <service-name> -> enter label name of the pod within the namespace

selector: 
  app: gw
  type: front-end 
 
k describe svc <service-name>

curl http://<cluster-IP:80>


vi svc.yaml  -> entering NodePort 

apiVersion: v1
kind: Service 
metadata:
  name: node-service  
spec:
  type: NodePort 
  ports:
    - targetPort: 80
	  port: 80	
      NodePort: 31000 
  selector: 
    app: gw
    type: front-end 
	  
	  
k create -f svc.yaml
k get svc 

===============================================================================
Taints -> Nodes 
Toleration -> Pod

k run usa-pod --image nginx
k get pod -o wide
k taint nodes node01 env=prod:NoSchedule
k describe nodes node01 
k describe nodes node01 | less 

k run ind-pod --image nginx
k get pod -o wide 
ind-pod wont be added in node01 coz of its already tainted

k edit pod ind-pod 
copy tolerations yaml code from k8s documentation


k run chn-pod --image nginx --dry-run=client -o yaml
k run chn-pod --image nginx --dry-run=client -o yaml > chn-pod.yaml
vi chn-pod.yaml
copy toleration yaml code inside under spec

k create -f chn-pod.yaml
k get pods -o wide 

k taint nodes node01 env=prod:NoSchedule-
The node will be untainted
k describe nodes node01 | grep -i taint 

Node Affinity ( Node )

k run first-pod --image nginx --dry-run=client -o yaml
k run first-pod --image nginx --dry-run=client -o yaml > first-pod.yaml

giving property inside spec 

affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: country 
            operator: In
            values:
            - ind 
			
k create -f first-pod.yaml
k get pods -o wide 
k describe pod first-pod

DaemonSet: 

k get ds
k get ds -n kube-system 

k create deployment ds1 --image k8s.gcr.io/fluentd-elasticsearch:1.20 -n kube-system --dry-run=client -o yaml 

k create deployment ds1 --image k8s.gcr.io/fluentd-elasticsearch:1.20 -n kube-system --dry-run=client -o yaml > ds1.yaml

vi ds1.yaml -> change kind name to DaemonSet
k create -f ds1.yaml 
k get ds -n kube-system 


Static pod:
The kubelet monitors this directory and automatically starts and stops the Pods based on changes to the manifest files. 
Static Pods are useful for running critical system components directly on a node, you can create a static Pod for that component to ensure it is always running on that node.

InitContainers:

k run dxb-pod --image nginx --dry-run=client -o yaml > init.yaml

vi init.yaml -> copy inityaml from k8s docs
k create -f init.yaml

Monitor cluster:

https://medium.com/@iamarunix/monitor-node-and-pod-using-metricserver-9ddae80bfc9b

k top pods 
k top pods -n kube-system 

Deployments Rollback:

k get deploy
k get deploy -n kube-system
k describe deploy metrics-server -n kube-system
StrategyType: RollingUpdate 

kubectl set image deployment.v1.apps/nginx-deployment nginx=nginx:1.16.1

k rollout status deployment <deployname>

k rollout history deployment <deployname>

Environmental variables:

Configmaps:
k get cm 
k create cm --help
k create configmap cm1 --from-literal=db=oracle --from-literal=cert=sso 
k get cm 
k run pod1 --image nginx --dry-run=client -o yaml
vi pod1.yaml- copy configmap doc template 
k create -f pod1.yaml 

Secrets:

----------
Volumes:

Temporary storage (empty Dir) -> ephemeral
Permanent storage (PV, PVC) -> persistence 

vi emptydir.yaml

copy empty dir from k8s doc 
==========================================================
Kubernetes End to End project on EKS | EKS Install and app deploy with Ingress:

https://www.youtube.com/watch?v=RRCrY12VY_s&t=25s

DevOps day to day activites | What and How GitOps works | Terraform for IaaS in Real-world:

https://www.youtube.com/watch?v=BJCkZuSW6ik

====================================================================================================================================

Abhishek veeramalla :

Master node + Worker node =Cluster

Controlplane: 
Kube-API server -> communication between master node components and worker node.
ETCD -> Inbuilt database, keep track of every cluster transaction that occurs. (Key value store)
Kube controller manager -> Monitor k8s objects and remediate the situation.
Kube Scheduler -> find best nodes for each containers.

Dataplane:
Kubelet -> Operates pods from API server instructions
Kubeproxy -> maintain network between nodes and enables comm between Pods and external APIs
Container Run time. 

Minikube & kubectl installation

vi pod.yaml 
k create -f pod.yaml 
k apply -f pod.yaml 
kubectl get pods
k get nodes 
k describe pod <podname>

k8s deployment 

containers -> created using docker run command
pod -> created through yaml manifest (running specification)-> single or multiple container 
deployment -> auto healing and auto scaling features. 

replicaSet -> k8s controller (proper healing feature) -> no of replica of pods 

k get deploy
k get rs
k delete deploy <deployname>

k8s Services: 

ideal pod count -> depends on the replica of application can handle.

Service discovery concept -> Labels and Selectors

label tag -> eg: app: payment

Service can expose to world. 

1. ClusterIP -> default , accessed inside cluster
2. nodeport -> allow application to access inside organization (Node Access)
3. Load balancer -> expose service to external world 

Docker-> Container platform
k8s -> cont orchestration env offers features like
Auto scaling, healing, Clustering and Enterprise level support like load balancing.

Namespace -> logical isolation of resources

Services types: ClusterIP mode, Nodeport mode, Loadbalancer mode 

k8s activities:
- manage k8s cluster for organization
- ensures the application are deployed on cluster
- troubleshooting issues on pods
- cont maintenace activities on worker nodes, upgrading the version, installing the mandatory packages on worker nodes
- resolving JIRA tickets based on issues 


k edit svc <svc name> 

Ingress: it exposes http and https routes from outside the cluster 
to services within cluster. traffic routing is controlled by rules 
defined on the ingress resource. 

k get ing 

Services doesn't have Enterprise level load balancer capabilties so they are using Ingress with service 

customer resource: 

CRD = org will defining new type of API to k8s -> submit the CRD to k8s 

config maps: 

========================================================================================

* Week 7

-> Continuous Integration/Continuous Delivery/Deployment (CI/CD)

-> Principles of CI/CD

-> Jenkins - Integration tool 

-> How to create jobs and pipelines in Jenkins 

-> How to integrate Jenkins with other tools

========================================================

CICD:

A continuous integration and continuous deployment (CI/CD) pipeline is a series of steps that must be performed in order to deliver 
a new version of software. 

CI/CD pipelines are a practice focused on improving software delivery throughout the software development life cycle via automation.

four major phases: source, build, test and deploy.

https://www.techtarget.com/searchsoftwarequality/CI-CD-pipelines-explained-Everything-you-need-to-know#:~:text=The%20CI%2FCD%20pipeline%20combines,%2C%20build%2C%20test%20and%20deploy.


Plan ->   Code ->  Build ->    Test   ->   Release ->  Deploy ->  Monitor 

Continuous Integration ===============      ========Continuous Delivery/Deploy ====

https://semaphoreci.com/cicd

github ->  Webhook -> Automation 

============================

Jenkins:

Intro

Installation

Creating jobs

Queuing of jobs 

-> Upstream and Downstream proj             A+B+C 

Dev

Prod

Test

Job -> configure -> Post-build Actions -> Build other projects -> proj name 

Build Triggers:

Build Trigger -> Build Periodically (crontab)

remote trigger :

Try to run it by giving URL 

=================

Ticketing tool: JIRA 

Dev - jenkins install for int purpose -> raise ticket -> devops team 

req = 1amazon linux ec2, 15gb ebs , t3,medium inst type, 22 port , 

---------------------------------------------------------------------

Security policy in jenkins! 

Plugins -> Role based authorization strategy 

Manage jenkins -> Security -> Authorization -> Role based strategy. (Manage and Assign Roles )

Create users -> User1 User2

Manage Jenkins -> manage and assign roles 

        global role- employee -> read permission        item role -> dev.* test.*   

 cron regex -> regex101.com 

Assign roles -> logout and login with particular users to check if there is access for given roles. 

-------------------------------------------------------------------

Tomcat Deployment in jenkins:

apache-tomcat-8.5.94  -> tar -xzvf <filename>

mv apache-tomcat-8.5.94 /opt/     -> /opt/ Add on software packages 
 

Jenkins and Tomcat port number - 8080

We need to change the port number - jenkins 

/opt/apache-tomcat-8.5.94/conf/ -> server.xml (config file)

configuration file path:

Ubuntu - /etc/default/jenkins
Amazon linux - /etc/sysconfig/jenkins

/etc/sysconfig/jenkins = changed port number to 9090 

service jenkins stop
service jenkins start 

https://www.youtube.com/watch?v=QYZ5Q6IqQ-s

systemctl stop jenkins
systemctl status jenkins

cd /etc/default/ 
ls 
vi jenkins                  change port number 9090 
cd /lib/systemd/system
ls
vi jenkins.service    -> change port number   - Environment="Jenkins_port=9090"
systemctl daemon-reload
systemctl restart jenkins 


/opt/apache-tomcat-8.5.94/ -> /bin/   -> startup.sh   shutdown.sh    Tomacat start or stop 

./startup.sh  -> Tomcat started

Deployment activity: 

Download sample war file 

cp /root/sample.war /var/lib/jenkins/workspace/autodeployment

Deploy to container plugin install 

/opt/apache-tomcat-8.5.94/conf -> vi tomcat-users.xml     

Add this line: 

<role rolename="manager-gui"/>
   <role rolename="manager-script"/>
   <role rolename="manager-jmx"/>
   <role rolename="manager-status"/>
   <user username="admin" password="admin" roles="manager-gui,manager-script,manager-jmx,manager-status"/>
   <user username="deployer" password="deployer" roles="manager-script"/>
   
Stop and Start Tomcat app
   
   
Error:


https://stackoverflow.com/questions/41675813/the-username-you-provided-is-not-allowed-to-use-the-text-based-tomcat-manager-e
   
   
===========================================================================

Declarative pipeline vs scripted pipeline.

Delivery pipeline
Build pipeline
Build Monitor



https://www.youtube.com/watch?v=RYQf5bHFEO0

https://www.youtube.com/watch?v=9RsmPNs7gT0    = #1 - Jenkins Master and Slave Configuration | How to run Jenkins job on Slave node

https://www.youtube.com/watch?v=Na282SnLmbQ     =  #2 - Build Maven project on Jenkins Slave Node 

https://www.youtube.com/watch?v=AgLn2xyFyTk    = #3 Jenkinsfile to Build and Push Image onto DockerHub


https://www.youtube.com/watch?v=hwrYURP4O2k  = Jenkins Master Slave Configuration | How to setup Jenkins master and slave

==============================




Pipeline:

Scripted Pipeline   = Not using 

Declarative pipeline   = Almost used 

---------------------

Dockerization using Nodejs application :

Code = nodejs 

COntainer tool  = Docker          Docker machine -> image -> 1 container  - httpd, tomcat, word press, 

SCM   = Github 

CI tool  = Jenkins 

Nodejs app:

package.json
server.js
DockerFile

---------------------------

git clone -> image build -> image run (container)

docker build -t nodejs .
docker run -itd --name=samplejs -p "9000:9000" nodejs
curl localhost:9000

(build)  docker images ->(run) comtainer 

pipeline {
    agent any

    stages {
        stage('Git Clone') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: 'new token', url: 'https://github.com/Shivoffline/samplenodejs.git']]])
                echo 'Project was cloned successfully'
            }
        }
        stage('docker stop') {
            steps {
                sh 'docker stop samplejs'
            }
        }
        stage('docker remove') {
            steps {
                sh 'docker rm samplejs'
            }
        }
        stage('docker remove image') {
            steps {
                sh 'docker rmi nodejs'
            }
        }
        stage('docker build') {
            steps {
                sh 'docker build -t nodejs .'
            }
        }
        stage('docker run') {
            steps {
                sh 'docker run -itd --name=samplejs -p "9000:9000" nodejs'
            }
        }
    }
}
		  
		  
================================================================================================

Sonar Qube setup on AWS :

https://www.youtube.com/watch?v=E5hMOGeBT-o&list=PLxzKY3wu0_FL3TzBnBeBoIMoRkXmYe3VB

https://github.com/ravdy/DevOps/blob/master/sonarqube/Setup_SonarQube.md

https://cigdemkadakoglu.medium.com/sonarqube-installation-on-ubuntu-20-04-with-community-branch-plugin-53e20cbded08

SonarQube Integration with Jenkins:

https://www.youtube.com/watch?v=wn9wWYAShag

===================================================================================================

Docker:

docker images
docker ps
docker ps -a

docker run -itd --name webserver -p "8888:80" httpd

docker exec -it <containerID>

docker stop contid

docker start contid

docker rm -f contname 
docker rmi imagename

docker inspect contid 
docker logs
docker stats
docker top

converting cont to image:

install vim git 

docker commit contname image1:v1.0 

backup:

docker save -o /usr/local/myimagebackup.tar image1:v1.0

docker load -i myimagebackup.tar 


docker login
docker tag image1:v1.0 shivoffline/httpd_vim_git
docker push shivoffline/httpd_vim_git


docker architecture:

Volumes:  backup or synchronization 

/root/html 

docker run -itd -p "7070:80" -v "/root/html:/usr/local/apache2/htdocs" httpd 

docker volume ls
docker volume create my-volume
docker inspect my-volume   - cd mountpoint path 
docker run -itd --mount source=my-vol,destination=/usr/local/apache2/htdocs -p "5050:80" httpd
docker run -itd --mount source=my-vol,destination=/usr/local/apache2/htdocs -p "6060:80" httpd

DockerFile

From centos
Label Name="Shiva"
Run yum install httpd -y

mkdir test

docker build -t myimage:v1.0 /root/test 

Now installing vim and git from DockerFile

FROM centos
LABEL Name="Shiva"
RUN yum install vim -y
RUN yum install git -y


sudo amazon-linux-extras install java-openjdk11 -y 

==================================

What is Jenkins, and how does it fit into the DevOps ecosystem?

Explain the difference between a Jenkins Pipeline and a Freestyle project. When would you choose one over the other?

How do you set up and configure Jenkins for the first time? Can you explain the key configuration options?

What is a Jenkinsfile, and how is it used in Jenkins pipelines?

How do you trigger a Jenkins job or pipeline automatically when changes are pushed to a Git repository?

Explain the concept of Jenkins agents or nodes. What is the difference between a master and a slave node?

What is the purpose of Jenkins Plugins, and can you name a few common Jenkins plugins used in real-world scenarios?

How do you secure a Jenkins server, and what are some best practices for Jenkins security?

Describe Blue-Green Deployment and Canary Deployment in the context of Jenkins. How can Jenkins facilitate these deployment strategies?

What are Jenkins build tools, and can you name some popular build tools integrated with Jenkins?

Explain the concept of parameterized builds in Jenkins. When and why would you use them?

How do you set up a multi-branch pipeline in Jenkins, and what are its benefits?

What is Jenkins Artifacts and how can you manage them in Jenkins?

Describe the use of the "archive" and "stash" steps in a Jenkins pipeline. When would you use one over the other?

How can you automate the deployment process using Jenkins, and what are some common deployment automation strategies?

Explain how to schedule builds and jobs in Jenkins. What is the purpose of the Jenkins Scheduler?

What is Jenkins Blue Ocean, and how does it enhance the Jenkins user interface and user experience?

Describe the key differences between Jenkins and other CI/CD tools like Travis CI, CircleCI, and GitLab CI/CD.

How do you handle Jenkins job failures, and what are some troubleshooting techniques you would use to identify and resolve issues?

Can you provide an example of a challenging real-world problem you've encountered while working with Jenkins and how you resolved it?




sudo amazon-linux-extras install java-openjdk11 -y 

which java

readlink -f <>
----------------------------------=========================================================================================

Week: 5 & 6

DevOps 

DevOps culture, principles, and practices

                     ============= I1==================== =========== I2 ==================== =========== I3 ================
Agile       =  Plan + code + Implementation + Deployment + code + Implementation + Deployment + code + Implementation + Deployment + Monitoring + go live

DevOps  =  Plan + code + Implementation + Deployment + Monitoring + Plan + code + Implementation + Deployment + Monitoring + Plan + code + Implementation + Deployment + Monitoring
           ============I1 ======================================== =====================I2 ===============================

DevOps = Agile + Open source Automation tools 

Quick delivery
Eliminates waste from project
Accuracy
Customer satisfaction / Business improvment

Engineering services:

1. Server Engineering 2. Database Engineering 3. Network Engineering 4. Application Engineering 5. Storage Engineering 6. Security Engineering


==============================================

DevOps Tools:

1. Ansible, Chef , Puppet                    - Deployment & Configuration mgmt tool 

2. GIT  (Source code management tool)        -  Version control system                        - 

3. Terraform                                 - Env Build tool  (IAC) Infra as a code 

4. Jenkins                                   - Integration tool 

6. K8S 										- container management tool - 

7. Docker									- container services 

10. DataDog, Splunk, AppD, Prometheus, Grafana. New Relic 


AWS - Cloud service - Infrastructure 

AWS - Infra
Devops - Implementation 

Devops = Bridge between Delivery business unit and Dev business unit 


========================================

Git - Version Control system

Github - Web based GIT repo 


Version control system = They are process management system while maintain changes recorded in a file or set of file over a period of time.

https://www.javatpoint.com/git-version-control-system

A version control system is a software that tracks changes to a file or set of files over time so that you can recall specific versions later. It also allows you to work together with other programmers.

The version control system is a collection of software tools that help a team to manage changes in a source code. It uses a special kind of database to keep track of every modification to the code.

Developers can compare earlier versions of the code with an older version to fix the mistakes.

Benefits:

Complete change history of the file
Simultaneously working
Branching and merging
Traceability

Types:

Localized version Control System:

The localized version control method is a common approach because of its simplicity. But this approach leads to a higher chance of error. In this approach, you may forget which directory you're in and accidentally write to the wrong file or copy over files you don't want to.

To deal with this issue, programmers developed local VCSs that had a simple database. Such databases kept all the changes to files under revision control. A local version control system keeps local copies of the files.

The major drawback of Local VCS is that it has a single point of failure.


Centralized version control systems:

The developers needed to collaborate with other developers on other systems. The localized version control system failed in this case. To deal with this problem, Centralized Version Control Systems were developed.

Centralized version control systems have many benefits, especially over local VCSs.

Everyone on the system has information about the work what others are doing on the project.
Administrators have control over other developers.
It is easier to deal with a centralized version control system than a localized version control system.
A local version control system facilitates with a server software component which stores and manages the different versions of the files.

Distributed version control systems:

Centralized Version Control System uses a central server to store all the database and team collaboration. But due to single point failure, which means the failure of the central server, developers do not prefer it. Next, the Distributed Version Control System is developed.

In a Distributed Version Control System (such as Git, Mercurial, Bazaar or Darcs), the user has a local copy of a repository. So, the clients don't just check out the latest snapshot of the files even they can fully mirror the repository. The local repository contains all the files and metadata present in the main repository.

====================================================

Git is a distributed version control system that enables software development teams to have multiple local copies of the project's codebase independent of each other. These copies, or branches, can be created, merged, and deleted quickly, empowering teams to experiment, with little compute cost, before merging into the main branch (sometimes referred to as the master branch). Git is known for its speed, workflow compatibility, and open source foundation.

Most Git actions only add data to the database, and Git makes it easy to undo changes during the three main states.

Git has three file states: Untracked/modified, staged, and committed.

1. A modified file has been changed but isn't committed to the database yet.

2.  staged file is set to go into the next commit.

3. When a file is committed, the data has been stored in the database.

With Git, software teams can experiment without fearing that they'll create lasting damage to the source code, because teams can always revert to a previous version if there are any problems.



========================================================
Creating GIThub account -> creating new repo 

Basic requirements:

1. Download GIT software for Windows. 
2. Download VSC 
3. Github sign in 

Command prompt: git config --global user.name "Shivoffline"
				git config --global user.email "sivaprasad1393@gmail.com"
				
GIT flow:

Working directory U = Local machine

Staging Area A = BUffer location/Parking area

LOcal repository = (.GIT) 

Remote repository = Github/gitlab/bitbucket 
				
Task 

1. Cloning file from github to VSC. -> git clone  (Clone entire repo)

    Created File2.txt in VSC -> git status  -> Untracked file U (Working Dir)
	
	git add File2.txt -> git status -> Staging area A 
	
	git commit -m "message"  -> Pushed to Local repo (.GIT)
	
	git push origin main -> Pushed to Remote repo (Github)
	

				Any File in VSC: 
				
				Insert/Modify -> Add -> Commit -> Push 
				
2.  In File2.txt , i have added a first line, so the status shows as M (Modified)
     so we need to follow the same process as above. 
	 
					Add -> Commit -> Push 
					
3. I have deleted File2.txt. you can view the deleted status using git status command

		now we need to make the changes in remote repo, so add and commit the changes to push to remote repo.
		
		Add File2.txt -> Commit -> Push main origin 
		
4. If we are making any changes in Github by creating or modifying any file, you can use git pull command to fetch all the changes. 

5. Created a new folder Tasknew with file name index.html

   git status won't work because git is not initialized in Tasknew folder. (.git folder is not inside Tasknew)
   
   git init  -> initialize git within Tasknew
   
   
6. Tried git push origin main command .. Doesn't works. so created a new repo with name Tasknew 

git remote push origin <link>

src doesn't match any.

git branch   -> *master            we need to change the branch name from master to main

git branch -M main                

now trying git push origin main command, index.html file will be moved to Tasknew github repo. 

git log 
git log --oneline   -> To check the commit details 

===================================

Branching & Merging: 

https://git-scm.com/book/en/v2/Git-Branching-Basic-Branching-and-Merging


First, i have created a branch1 (Feature branch)

git pull command 

git branch <newbranchname>   -> To create a new branch 

git branch -a   -> command shows branch details   

git checkout branch1   ->   * branch1 (To switch between branches)

Then, i have created a new file named branch1.txt and pushed to Remote repo

git push origin branch1 

Anyway, the changes wont reflect in main branch. 

branch1.txt file will be visible only in branch1   -> You need to merge feature branch with main branch 

git merge branch1    -> git push origin main 

Merge - Wont directly merge Feature branch with main branch -> Pull Request 


main

branch1, branch2, branch3 ------ branchn = Feature branch 

===================

Pull Request: 

https://www.atlassian.com/git/tutorials/making-a-pull-request


Merge conflicts:

https://opensource.com/article/23/4/resolve-git-merge-conflicts

https://www.javatpoint.com/git-merge-and-merge-conflict

==========================

Working with remote repositories:

https://git-scm.com/book/en/v2/Git-Basics-Working-with-Remotes

=============================

Git commands:

git clone
git log / git log --oneline
git status
git add
git commit
git push
git pull
git fork
git stash
git rebase  - The git rebase command allows you to easily change a series of commits, modifying the history of your repository
git cherrypick  - git cherry-pick in git means choosing a commit from one branch and applying it to another branch
git fetch
git remote add

=====================================================================================================================

