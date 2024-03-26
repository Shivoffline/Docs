# Docs

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

Shell scripting:

#! -> Shebang 
/bash -> executable used to execute the shell script 
#!/bin/bash -> default used syntax

man command 

chmod -> grand permissions

User, Group, Everyone 

4-> read
2-> write
1-> execute 

chmod 777 

commands:
History  
nproc
free 
top
df -h
ps -ef 

Instead of "echo" use "set -x"
set -x  -> debug mode 

date | echo "date is"

Output:  date is

stdin stdout - channels or log flows in VMs
date is a system default command.
It sends the output to the pipe will not be able to receive the information from stdin.
If the command is not ready to send the info to stdin.  

===============================

set -e   -> It exits the script when there is error

set -o pipefail   -> exits the script if an error while using pipe 

logfile   -> find errors in logfile 

curl  -> retrieve information from any website/link.
         transfer URL 
		 
		 curl <URL>
		 
enables data exchange between a device and a server through a terminal.

wget -> download URL file 

trap -> trapping signals 

================
1. To list all processes:

awk -> filter the output of a particular string
ps -ef | awk -F" '{print $4}'

2. Print only errors from a remote log:

curl <link> | grep TRACE 

3. Print numbers divided by 3 & 5 and not 15:

divisible by 3, div by 5, not 3*5=15

for i in {1..100}; do 
if ([ expr $i % 3 == 0 ] || [ expr $i % 5 == 0 ])) && [ expr $i % 15 != 0 ];
then
		echo $i
fi;
done

for ((i = 1; i <= 100; i++)); do
    # Check if the number is divisible by 3 and 5 but not 15
    if ((i % 3 == 0)) && ((i % 5 == 0)) && ((i % 15 != 0)); then
        echo $i
    fi
done

4. print number of "S" in "mississippi"

x=mississipi

grep -o "s" <<<"$x" | wc -l 

5. How to debug the shell script:

set -x

6. Crontab in linux and its usage:

It acts like an alarm to set up in linux for execution for any process/task.

7. How to open a read only file:

vim -r test.txt

8. Diff Soft and Hard link:

A hard link always points a filename to data on a storage device. 
A soft link always points a filename to another filename, which then points to information on a storage device.

9. Diff Break and continue statement:

Break -> Breaking the execution
Continue -> Continuing the execution (Skip)

10. Disadvantages of Shell scripting:

- Execution speed is slow

11. Different types of loops:

While
For
Until

12. Is Bash dynamic or Statically typed?

Dynamically typed. 

In dynamically typed languages, the type of a variable is interpreted at runtime, and the type of a variable can change during the execution of the program.

13. Network Troubleshooting utilty:

traceroute google.com 

14. Sort list of files:

sort command

15. Manage logs of a sys generate huge log files everyday:

logrotate 

=========================================================
Terraform:


Providers
Multi-Region
Multi Cloud
Variables and conditions 

Modules
Statefile  -> terraform show 
Remote backend 
provisioners
file Provisioner, remote-exec Provisioner, local-exec Provisioner

==========================================================================

Kubernetes CGI Int topics:

K8s, kubespray, kubeadm, Sidecar containers, Init containers, Static pods and purpose, Stateful Set,
Daemon set, Services types, Pod issue, restrict pod communication, Pod & PVC, Deployment rolling strategy
Diff branching strategies, git stash, git rebase
Helm charts

==========================================================================

