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

