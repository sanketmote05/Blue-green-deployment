Implementing blue-green deployment in Amazon EKS (Elastic Kubernetes Service) involves creating two separate environments (the blue and green environments) that allow for seamless switching between application versions. Here’s a general approach to implementing it:

Prerequisites

An EKS cluster.

kubectl configured to communicate with your EKS cluster.

A containerized application to deploy.


Steps for Blue-Green Deployment

1. Define the Blue and Green Deployments

Create two Kubernetes deployments, one for each environment (e.g., blue-deployment and green-deployment).

Each deployment should use a different container image version (e.g., app:v1 for blue and app:v2 for green) to represent the two versions of your application.


Example YAML (blue-deployment.yaml):

apiVersion: apps/v1
kind: Deployment
metadata:
  name: blue-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
      version: blue
  template:
    metadata:
      labels:
        app: my-app
        version: blue
    spec:
      containers:
      - name: my-app
        image: my-app:v1
        ports:
        - containerPort: 80

Example YAML (green-deployment.yaml):

apiVersion: apps/v1
kind: Deployment
metadata:
  name: green-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
      version: green
  template:
    metadata:
      labels:
        app: my-app
        version: green
    spec:
      containers:
      - name: my-app
        image: my-app:v2
        ports:
        - containerPort: 80

2. Create a Service for Traffic Routing

Use a Kubernetes Service of type LoadBalancer or ClusterIP that will route traffic between the blue and green environments.

Initially, set the Service selector to match the blue deployment's labels.


Example YAML (service.yaml):

apiVersion: v1
kind: Service
metadata:
  name: my-app-service
spec:
  selector:
    app: my-app
    version: blue  # Initially set to blue
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: LoadBalancer

3. Test the Green Deployment

Before switching traffic, ensure the green deployment (new version) is working as expected.

You can access it directly by using a temporary service or port-forwarding to check the green deployment.


Temporary Green Service (Optional for Testing):

apiVersion: v1
kind: Service
metadata:
  name: green-service
spec:
  selector:
    app: my-app
    version: green
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80

4. Shift Traffic to the Green Deployment

Update the selector in my-app-service to point to the green version.

Apply the change, which will direct all traffic to the green environment.


# Update my-app-service to point to green
spec:
  selector:
    app: my-app
    version: green

Apply the changes:

kubectl apply -f service.yaml

5. Monitor and Rollback if Needed

Monitor the application’s behavior in the green environment to ensure everything is stable.

If issues are detected, revert the service selector back to blue to roll back traffic to the previous version.


# Revert my-app-service to point back to blue
spec:
  selector:
    app: my-app
    version: blue

6. Clean Up and Scale Down Blue Deployment

After successfully verifying the green deployment, scale down or remove the blue deployment to free up resources.


kubectl scale deployment blue-deployment --replicas=0

Optional: Automate with ArgoCD or Helm

For added automation and ease of deployment:

Use ArgoCD or Helm to manage the versioning and configuration, making rollback or forward changes faster.

Use Ingress controllers for better routing and traffic control.


Best Practices

Use health checks and readiness probes to ensure the application’s stability before routing traffic.

Implement monitoring and logging to quickly detect any issues after the deployment switch.

Perform canary testing alongside blue-green if a gradual rollout is needed.


This approach allows for a controlled transition between versions with minimal downtime in your EKS environment.
