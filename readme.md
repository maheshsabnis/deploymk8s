# Install Microk8s

#Install MicroK8s on Linux
sudo snap install microk8s --classic

Don’t have the snap command? Get set up for snaps

#Check the status while Kubernetes starts
microk8s status --wait-ready

#Turn on the services you want
microk8s enable dashboard dns registry istio

# dashboard: the addon that will be used to show the Kubernetes management page in browser
# dn: Deploy the CoreDNS to supply an address resolution to Kunernetes on Microk8s
# THis will offer IP address resilution to various services runing

# Command to create a new namespace

microk8s kubectl create ns mynamespace

#Try microk8s enable --help for a list of available services and optional features. 
microk8s disable <name> turns off a service.


# Display list of all namespaces
# namespaces: These are used to group Microservices deployed in Kubernetes
microk8s kubectl get all --all-namespaces

# To show the Dashboard, we need to access the Dashboard service by logging on it using token
# to receive the token, run the following command
token=$(microk8s kubectl -n kube-system get secret | grep default-token | cut -d " " -f1)

- kube-system: is the namespace generated after installing MIcrok8s
# Assign the secret to kubectl so that the login can takes place

microk8s kubectl -n kube-system describe secret $token

# Create a Deployment and Servce configuration using '.yml' file
deployment.yml

```javascript
apiVersion: apps/v1
kind: Deployment
metadata:
  name: aspnet-catapi-deployment # the deplpoyment name on clusrter
  namespace: default
  labels:
    app: aspnet-catapi-deployment
spec:
  selector:
    matchLabels:
      app: asptnet-catapi-pod # the pod name that will contain container
  template: # confoguration for deployment
    metadata:
      labels:
        app: asptnet-catapi-pod
    spec:
      containers:
        - name: aspnet-catapi-container # the container name in POD
          image: mast007/catapi:v1 # image name that will be pulled
          resources: # very important configuration for the POD so that service will be loaded and executed
            limits:
              cpu: "500m" # allocate half CPU for this service
              memory: "128Mi" # 128 MB of memory
          ports:
            - containerPort: 80            
                  
      
```

The service.yml

```javascript
apiVersion: v1
kind: Service
metadata:
  labels:
    app: aspnet-catapi # the service app name
  name: aspnet-catapi-service # name for the service deployment
spec:
  ports:
    - port: 8080 # the port used to call service from external apps
      targetPort: 80 # the container port
  selector:
    app: asptnet-catapi-pod # the POD where the image will be deployed in container
  type: LoadBalancer # provides a public access to the service        
```

# Applying yml for deployment on Kubernetes

microk8s kubectl apply -f deployment.yaml

microk8s kubectl apply -f service.yaml

# List all pods
microk8s kubectl get pods
# List all services
microk8s kubectl get services / microk8s kubectl get svc
# To describe a pod to see if the image is successfully pulled from repository
microk8s kubectl describe pods <POD-NAME>
# To delete the service and deployment from Kubernetes run the following command
microk8s kubectl delete -f service.yml
microk8s kubectl delete -f deployment.yml


# The Microk8s will provide the service details as followes
NAME                    TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
aspnet-catapi-service   LoadBalancer   10.152.183.134   <pending>     8080:32201/TCP   2m49s

- NAME: is service name
- TYPE: The Access of the service
- CLUSTER-IP: The internal IP of the cluster
- EXTERNAL-AP: The IP provided by the Cluster to access service from external / puvlic clients
    - NOTE: Since Microk8s is virual env. the service will ne be allocated with External IP. You will get external IP in AKS, EKS, GKS
- PORT(S): the Port mapping of Container exposed port  (defined in service.yml)  for external communication will be mapped with the Microk8s VM port
    - 8080/32201/TCP
        - 8080 is a port used for publically accessing the Service and 32201 is a port exposed by the Microk8s to that service can be accessed using 
            - http://locahost:32201/api/[Controller]
            - OR
            - http://Kubernetes-Dashboard-IP:32201/api/[Controller] (Not receommended)


