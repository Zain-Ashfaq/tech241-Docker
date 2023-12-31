# tech241-Docker

# Docker Setup and Installation

To use Docker on your system, follow the setup instructions based on your operating system:

## Windows

1. **Docker WSL2/Linux subsystem on Windows:**
   - Set up Docker to work with Windows Subsystem for Linux 2 (WSL2).
   - Click [here](https://docker.com/setup/windows) for the Docker Desktop installation on Windows.

2. **Docker Desktop:**
   - Install Docker Desktop for Windows by clicking [here](https://docker.com/setup/windows).
   - Make sure to enable Windows Hyper-V settings for Docker to function properly.

Please note that Docker on Windows provides two options: using WSL2/Linux subsystem or installing Docker Desktop directly. Choose the one that suits your needs best.
# Docker API Overview

The Docker API is an HTTP API that enables remote interaction with the Docker daemon. It provides programmable control over Docker containers, images, networks, and other resources. The API offers a set of endpoints for performing various actions.

## Main Components and Functionalities

1. **Docker Daemon:** The Docker daemon is a background service responsible for managing Docker objects like containers, images, and volumes. It exposes an HTTP RESTful API for communication.

2. **API Endpoints:** The Docker API provides RESTful endpoints for specific Docker operations, such as creating containers, listing containers, inspecting images, etc. Common endpoints include:

   - `/containers`: Interact with containers (e.g., create, start, stop, remove).
   - `/images`: Work with Docker images (e.g., pull, build, remove).
   - `/networks`: Manage Docker networks.
   - `/volumes`: Control Docker volumes.

3. **HTTP Requests:** To interact with the Docker API, you make standard HTTP requests (GET, POST, PUT, DELETE) to the appropriate endpoints. Tools or libraries capable of making HTTP requests, like cURL, Python requests, or Node.js libraries, are commonly used.

4. **JSON Format:** Data exchanged with the Docker API is usually in JSON format, including request bodies and responses. When creating or modifying Docker resources, you need to send the necessary data in the request body.

5. **Authentication:** Depending on your Docker setup, authentication credentials might be required when accessing the API to ensure security.

## Running a Docker Image

To run a Docker image, use the following command:

```bash
docker run hello-world
```
Upon running the image, Docker will check if the image is available locally. If not, it will fetch the image from the online registry.

## Example Workflow

```bash
docker ps                # Shows a list of running containers
docker ps -a             # Shows a list of all containers
docker run -d -p 80:80 nginx   # Runs the nginx image with port forwarding
docker stop <docker_id>   # Stops the specified container
docker start <docker_id>  # Starts the specified container
docker exec -it <container_id> sh   # Communicates with the container via shell
```
## Saving Changes to Docker Image and Pushing to Docker Hub
To save changes made to a Docker image and push the updated image to a Docker repository, follow these steps:

## Make Changes to the Container: Modify files or install software in the running Docker container.

## Commit Changes to a New Image: Create a new image from the container using the docker commit command:

```bash

docker commit <container_id> <new_image_name>:<tag>
```
Tag the Image for the Repository: Tag the new image with the appropriate name and tag that matches your Docker repository:

```bash

docker tag <new_image_name>:<tag> username/repository:tag
```
Push the Image to Docker Hub: Push the updated image to Docker Hub using the docker push command:

```bash

docker push username/repository:tag
```
# Creating K8 Deployment for NGINX:
```yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
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
        image: zain453/tech241-nginx
        ports:
        - containerPort: 80
```
## Creating K8 Deployment for NODE:
```yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: node-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: node
  template:
    metadata:
      labels:
        app: node
    spec:
      containers:
      - name: node
        image: zain453/tech241-node-app
        ports:
        - containerPort: 3000
```
## Creating NGINX Service:
```yaml

apiVersion: v1
kind: Service
metadata:
  name: nginx-svc
  namespace: default
spec:
  ports:
  - nodePort: 30001
    port: 80
    targetPort: 80
  selector:
    app: nginx
  type: NodePort
Creating NODE Service:
yaml
Copy code
apiVersion: v1
kind: Service
metadata:
  name: node-svc
  namespace: default
spec:
  ports:
  - nodePort: 30002
    port: 3000
    targetPort: 3000
  selector:
    app: node
  type: NodePort
```
After writing the YAML files for each service, you can create the services using the following commands:

```bash

kubectl create -f nginx-k8.yml  # Creating the NGINX deployment
kubectl create -f node-k8.yml   # Creating the NODE deployment
kubectl create -f nginx-service.yml  # Creating the NGINX service
kubectl create -f node-service.yml   # Creating the NODE service
```
You can check the status of the deployments and services using the following commands:

```bash



kubectl get deployment  # Check running deployments
kubectl get pods       # Check running pods
kubectl get svc        # Check services and their information
```

# Connecting app to MongoDB

## Step 1: Create PVC for MongoDB

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mongo-db
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 256Mi
```
## To create the PVC, run the following command:

```bash

kubectl create -f mongodb-pvc.yml
Step 2: Create Mongo Deployment
yaml
Copy code
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongo
spec:
  selector:
    matchLabels:
      app: mongo
  replicas: 1
  template:
    metadata:
      labels:
        app: mongo
    spec:
      containers:
        - name: mongo
          image: zain453/mongodb
          ports:
            - containerPort: 27017
          volumeMounts:
            - name: storage
              mountPath: /data/db
      volumes:
        - name: storage
          persistentVolumeClaim:
            claimName: mongo-db
```
## Step 3: Create Mongo Service
```yaml

apiVersion: v1
kind: Service
metadata:
  name: mongo
spec:
  selector:
    app: mongo
  ports:
    - port: 27017
      targetPort: 27017
```
## Step 4: Create Node Deployment
```yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: node
spec:
  selector:
    matchLabels:
      app: node
  replicas: 3
  template:
    metadata:
      labels:
        app: node
    spec:
      containers:
        - name: node
          image: zain453/tech241-sparta-app
          ports:
            - containerPort: 3000
          env:
            - name: DB_HOST
              value: mongodb://mongo:27017/posts
          imagePullPolicy: Always
```
## Step 5: Create Node Service
```yaml

apiVersion: v1
kind: Service
metadata:
  name: node-svc
  namespace: default
spec:
  ports:
    - nodePort: 30002 # range is 30000 - 32768
      port: 3000
      targetPort: 3000
  selector:
    app: node
  type: NodePort # also use LoadBalancer - for local use ClusterIP
```
# Editing Deployments, Horizontal Pod Autoscaler, and Persistent Volume, Kubernetes Deployment Editing
To edit and manage deployments, you can use the following kubectl command:

```shell

kubectl edit deployment <name_of_deployment>
```
Horizontal Pod Autoscaler (HPA) Configuration
To configure Horizontal Pod Autoscaler for a deployment, create an HPA manifest as follows:

```yaml

apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  name: node-hpa
  namespace: default
spec:
  maxReplicas: 9
  minReplicas: 3
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: node
  targetCPUUtilizationPercentage: 50
```
Persistent Volume (PV) and Persistent Volume Claim (PVC) in Kubernetes
In Kubernetes, PV and PVC work together to provide storage capabilities to your applications. PV is a storage resource, while PVC is a request for storage.

When you need storage space for your application, Kubernetes can automatically find or create the right storage resource based on your PVC requirements.

Here's an example of creating PV and PVC:

Create PV:
```yaml

apiVersion: v1
kind: PersistentVolume
metadata:
  finalizers:
  - kubernetes.io/pv-protection
  labels:
    type: local
  name: node-pv 
spec:
  accessModes:
  - ReadWriteOnce
  capacity:
    storage: 1Gi
  hostPath:
    path: /tmp/data
    type: ""
  persistentVolumeReclaimPolicy: Retain
  volumeMode: Filesystem
  storageClassName: manual
Create PVC:
yaml
Copy code
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: node-pvc
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 512Mi
  storageClassName: manual
```
Node Deployment Example
To deploy an application with the PV and PVC, update your deployment manifest like this:

```yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: node
spec:
  selector:
    matchLabels:
      app: node
  replicas: 3
  template:
    metadata:
      labels:
        app: node
    spec:
      volumes:
        - name: node-pv
          persistentVolumeClaim:
            claimName: node-pvc
      containers:
        - name: node
          image: zain453/tech241-node-app
          ports:
            - containerPort: 3000
          volumeMounts:
            - name: node-pv
              mountPath: /tmp/data
          env:
            - name: DB_HOST
              value: mongodb://mongo:27017/posts
          imagePullPolicy: Always
```
Apply the deployment manifest using the following command:

```shell

kubectl apply -f <name_of_the_deployment_file>
```
To check the status of PVCs, you can use:

```shell

kubectl get pvc
```
This will show the names, statuses, capacities, access modes, and other relevant information of the PVCs in the cluster.
# Mongodb whole script
```yaml
# Combined YAML file for multiple Kubernetes objects

apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  name: sparta-mongo-db-deploy
  namespace: default
spec:
  maxReplicas: 9
  minReplicas: 3
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: mongo
  targetCPUUtilizationPercentage: 50

---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mongo-pv
  namespace: default
  labels:
    app: mongo
    type: data
spec:
  capacity:
    storage: 512Mi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /data/mongo-data # Replace this with the desired host path
  volumeMode: Filesystem
  persistentVolumeReclaimPolicy: Retain

---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mongo-db
  namespace: default
  labels:
    app: mongo
    type: data
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 256Mi

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongo
  namespace: default
  labels:
    app: mongo
spec:
  selector:
    matchLabels:
      app: mongo
  replicas: 1
  template:
    metadata:
      labels:
        app: mongo
    spec:
      containers:
        - name: mongo
          image: zain453/mongodb
          ports:
            - containerPort: 27017
          resources: # Add resource and limit specifications here
            requests:
              memory: "256Mi" # Minimum memory required
              cpu: "200m"     # 200 milli CPUs (0.2 CPU)
            limits:
              memory: "512Mi" # Maximum memory allowed
              cpu: "1"        # 1 CPU (1000 milli CPUs)
          volumeMounts:
            - name: storage
              mountPath: /data/db
      volumes:
        - name: storage
          persistentVolumeClaim:
            claimName: mongo-db

---
apiVersion: v1
kind: Service
metadata:
  name: mongo
  namespace: default
  labels:
    app: mongo
spec:
  selector:
    app: mongo
  ports:
    - port: 27017
      targetPort: 27017
```
Run these commands and this should appear in the terminal.
```
$ kubectl create -f mongo-whole-script.yml

persistentvolume/mongo-pv unchanged
persistentvolumeclaim/mongo-db created
deployment.apps/mongo created
service/mongo created
```
