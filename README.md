# Coworking Space Service 
The Coworking Space Service is a set of APIs that enables users to request one-time tokens and administrators to authorize access to a coworking space.

## Getting Started

### Dependencies
#### Local Environment
1. Python Environment - run Python 3.6+ applications and install Python dependencies via `pip`
2. Docker CLI - build and run Docker images locally
3. `kubectl` - run commands against a Kubernetes cluster
4. `helm` - apply Helm Charts to a Kubernetes cluster

#### Remote Resources
1. AWS CodeBuild - build Docker images remotely
2. AWS ECR - host Docker images
3. Kubernetes Environment with AWS EKS - run applications in k8s
4. AWS CloudWatch - monitor activity and logs in EKS
5. GitHub - pull and clone code

## Setup

#### 1. Create EKS Cluster

1. Open EKS console and create a EKS cluster.
2. Waiting for EKS cluster created, navigate to Compute tab and create Node Group.
3. Update the Kubeconfig
```
aws eks --region <region> update-kubeconfig --name <your_cluster_name>
```

#### 2. Configure a Database

Set up a Postgres database using a Helm Chart.

1. Prepare storage for database.
```
kubectl apply -f deployment/pv.yaml
kubectl apply -f deployment/pvc.yaml
```

2. Set up Bitnami Repo
```
helm repo add bitnami https://charts.bitnami.com/bitnami
```

3. Install PostgreSQL Helm Chart
```bash
helm install coworking-db bitnami/postgresql `
  --set auth.username=admin `
  --set auth.password=admin123 `
  --set auth.database=coworking `
  --set primary.persistence.existingClaim=coworking-space-pvc `
  --set volumePermissions.enabled=true
```

4. Test Database Connection
* Connecting Via Port Forwarding
```bash
kubectl port-forward --namespace default svc/coworking-db-postgresql 5432:5432 &
psql --host 127.0.0.1 -U admin -d coworking -p 5432
```

5. Run Seed Files
We will need to run the seed files in `db/` in order to create the tables and populate them with data.

* On Windows:
```bash
kubectl port-forward --namespace default svc/<SERVICE_NAME>-postgresql 5432:5432
psql --host 127.0.0.1 -U postgres -d postgres -p 5432 -f <FILE_NAME.sql>
```

#### 3. Build Analytics Application Using AWS CodeBuild

1. Push Analytics Application source code and CodeBuild template to your Github repository.
2. Create CodeBuild project and config project to run your CodeBuild template.

#### 4. Deploy the Analytics Application to your EKS cluster

1. Open ECR console and find the Analytics application image's URI
2. Update your deployment template
```Yaml
containers:
  - name: coworking-space
    image: 346395109265.dkr.ecr.us-east-1.amazonaws.com/coworking-space:1
```
3. Deploy your application
```bash
kubectl apply -f deployment/db-secret.yaml
kubectl apply -f deployment/db-configmap.yaml
kubectl apply -f deployment/coworking-space-configmap.yaml
kubectl apply -f deployment/coworking-space.yaml
```

## Monitor

### 1. Install Amazon CloudWatch Observability Add-On

* With Amazon CloudWatch cross-account observability, you can monitor and troubleshoot applications that span multiple accounts within a Region.
* Seamlessly search, visualize, and analyze your metrics, logs, traces, and Application Insights applications across linked accounts without account boundaries1.

## Troubleshoot

### 1. Check cluster information:
```
kubectl cluster-info
```
This command displays information about the Kubernetes cluster, including the cluster endpoint and the Kubernetes version.

### 2. Check nodes:
```
kubectl get nodes
```
This command lists all the nodes in the cluster and their status. It can help you verify if the nodes are running and ready.

### 3. Check pods:
```
kubectl get pods --all-namespaces
```
This command lists all the pods in all namespaces. It can help you identify if any pods are not running or experiencing issues.

### 4. Describe a pod:
```
kubectl describe pod <pod-name> -n <namespace>
```
This command provides detailed information about a specific pod, including its current state, events, and any error messages. Replace *pod-name* with the name of the pod and *namespace* with the namespace it belongs to.

### 5. Check logs of a pod:
```
kubectl logs <pod-name> -n <namespace>
```
This command displays the logs of a specific pod. It can help you troubleshoot issues by examining the output and error messages. Replace *pod-name* with the name of the pod and *namespace* with the namespace it belongs to.

### 6. Execute a command in a pod:
```
kubectl exec -it <pod-name> -n <namespace> -- <command>
```
This command allows you to execute a command directly inside a pod. It can be useful for troubleshooting or debugging purposes. Replace *pod-name* with the name of the pod, *namespace* with the namespace it belongs to, and *command* with the desired command.
