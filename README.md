# Minikube Tutorial

## Table of Contents

- [Chapter 1 - Installing Minikube](#chapter-1---installing-minikube)
- [Chapter 2 - Configuring Minikube](#chapter-2---configuring-minikube) 
- [Chapter 3 - Interacting with Your Cluster](#chapter-3---interacting-with-your-cluster)
- [Chapter 4 - Deploying Applications](#chapter-4---deploying-applications)
- [Chapter 5 - Addons and Integrations](#chapter-5---addons-and-integrations)
- [Chapter 6 - Tips and Tricks](#chapter-6---tips-and-tricks)
- [Chapter 7 - Conclusion](#chapter-7---conclusion)

## Chapter 1 - Installing Minikube

Minikube allows you to quickly set up a local Kubernetes cluster on your workstation. This chapter will walk through downloading, installing, and configuring minikube on different platforms like Linux, macOS, and Windows.

### Downloading Minikube

The first step is to download the latest minikube binary from the releases page: https://github.com/kubernetes/minikube/releases

For example, to download version 1.25.2 on Linux run:

```
curl -LO https://storage.googleapis.com/minikube/releases/v1.25.2/minikube-linux-amd64
```

Store the binary in a location like /usr/local/bin: 

```
sudo install minikube-linux-amd64 /usr/local/bin/minikube
```

Do the same process for Windows and macOS using the appropriate minikube-windows-amd64.exe or minikube-darwin-amd64 binary. 

Verify the installation:

```
minikube version
```

### Installing a Hypervisor

Minikube requires a hypervisor to run the Kubernetes cluster. We'll cover the steps to install VirtualBox, the default driver.

#### Linux

Install VirtualBox using your package manager:

```
sudo apt install virtualbox // Ubuntu
sudo dnf install virtualbox // Fedora
```

#### Windows

Download the VirtualBox installer (.exe) from https://www.virtualbox.org/ and run it.

#### macOS

Install VirtualBox from https://www.virtualbox.org/ or via Homebrew:

```  
brew install --cask virtualbox
```

Now minikube can use VirtualBox to provision Kubernetes. Next we'll cover starting and configuring minikube.

## Chapter 2 - Configuring Minikube 

With minikube installed, we can now configure our local Kubernetes cluster. This involves selecting the VM driver, allocating resources, and customizing the cluster setup.

### Choosing a VM Driver

By default minikube uses VirtualBox but supports other drivers like VMware, Docker, etc. To use an alternative driver:

```
minikube config set driver vmware 
```

Verify the driver with: 

```
minikube config view
```

### Allocating Resources 

To improve performance, we can allocate more CPUs and memory to minikube:

```
minikube config set cpus 4
minikube config set memory 8192 
```

This allocates 4 CPUs and 8GB RAM. Adjust as per your workstation hardware.

### Kubernetes Version

To target a specific Kubernetes release:

```
minikube config set kubernetes-version v1.23.3
```

You can also use version aliases like `stable`, `latest`, etc. 

### Cluster Name 

To set a custom cluster name: 

```
minikube config set cluster-name <name>
```

The name will appear in kubectl and on the dashboard.

### Other Customizations

Further customizations like network plugins, container runtime, etc. can be made via: 

```
minikube config set <option> <value>
```

Now we have a customized minikube cluster configured! Let's start it up.

## Chapter 3 - Interacting with Your Cluster 

With minikube installed and configured, we can now start up our local Kubernetes cluster and interact with it.

### Starting Minikube

Start the cluster using:

```
minikube start
```

This will provision the VM, Kubernetes components, and addons. It may take a few minutes the first time.  

Verify the cluster status with:

```
minikube status
```

The output should show the cluster name, Kubernetes version, VM driver, etc.

### Kubectl

The kubectl command line tool lets you manage the cluster and resources. Try listing nodes:

```
kubectl get nodes
```

This should show the single minikube node ready.

### Dashboard 

To view the graphical Kubernetes dashboard:

```
minikube dashboard 
```

This will automatically open the dashboard webpage. Try creating a sample deployment via the UI.

### Services

To access a load balancer service on minikube:

```
minikube service <service-name> 
```

This will route to the service IP, allowing you to test applications. 

### SSH

To ssh into the VM instance:

```
minikube ssh
```

From here you can interact with containers, run debugging commands etc.

Now we're ready to deploy applications on minikube!

## Chapter 4 - Deploying Applications

A key benefit of minikube is being able to quickly deploy and test applications in a local Kubernetes environment. In this chapter, we'll walk through deploying a simple Node.js app.

### Building Images 

First, write a simple Node.js app:

```js
// server.js
const http = require('http');

const requestHandler = (req, res) => {
  res.end('Hello World!'); 
}

const server = http.createServer(requestHandler);

server.listen(8080, () => {
  console.log('Server listening on port 8080');  
});
```

Next, create a Dockerfile:  

```dockerfile
FROM node:16-alpine
WORKDIR /app
COPY server.js .  
CMD ["node", "server.js"]   
```

Build the image:

```
docker build -t my-node-app .
```

### Pushing to Registry 

Tag and push the image to Docker Hub or a private container registry.

```
docker tag my-node-app <username>/my-node-app:v1  
docker push <username>/my-node-app:v1
```

### Creating Deployments

Write a deployment manifest:

```yaml 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-node-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-node-app
  template:
    metadata:
      labels:
        app: my-node-app
    spec:
      containers:
      - name: my-node-app
        image: <username>/my-node-app:v1
        ports:
        - containerPort: 8080
```

Deploy to minikube:

```
kubectl apply -f deployment.yaml
```

Verify pods are running:

```  
kubectl get pods
```

You should see 3 pods for your deployment. 

### Creating a Service 

Expose the deployment via a service:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-node-service
spec:
  type: LoadBalancer
  selector:
    app: my-node-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
```

Apply it:

```
kubectl apply -f service.yaml
```

You can now access your app via the minikube IP!

```
minikube service my-node-service
```

This covers the end-to-end workflow for deploying a simple app on minikube. Continue exploring adding ingresses, configmaps, and more.

## Chapter 5 - Addons and Integrations

Minikube includes a set of built-in addons that provide additional Kubernetes functionality. These addons allow you to enable ingress, container registries, monitoring, and more.

### Ingress

The nginx ingress addon allows ingress controllers to be deployed:

```  
minikube addons enable ingress
```

This can be used to route external traffic to services.

### Container Registry 

Minikube can deploy an in-cluster Docker registry:

```
minikube addons enable registry
```

Images can then be pushed to the registry IP/port instead of an external repository.

### Metrics Server

The metrics server collects resource usage data for nodes and pods:

```
minikube addons enable metrics-server
```

This enables the metrics API that Horizontal Pod Autoscalers use.

### Monitoring

Prometheus and Grafana can be deployed for monitoring:

```
minikube addons enable prometheus
minikube addons enable grafana
```

Grafana will be available on a NodePort service.

### Service Mesh  

To experiment with a service mesh like Istio:

```
minikube addons enable istio 
```

This allows you to play with Istio sidecar injection and traffic management.

### Storage

Addons like hostpath-provisioner allow testing persistent storage:

```
minikube addons enable hostpath-provisioner
```

Explore other addons like Helm, LogViewer, and more for extended minikube functionality.

## Chapter 6 - Tips and Tricks

After working with minikube for a while, here are some useful tips and best practices to get the most out of it.

### Stopping and Deleting

To stop the cluster:

```
minikube stop
```

To delete it completely: 

```
minikube delete
```

Stop avoids re-downloading images on next start. Delete does a clean shutdown.

### Caching Images  

Use a host Docker daemon to avoid re-pulling images:

```
minikube config set docker-env HTTP_PROXY=http://proxy-ip:port  
```

Or specify a custom image repository:

```
minikube config set image-repository my.registry.com:5000
```

### File Mounting

Mount a host folder to persist files across restarts:

```
minikube mount <source>:<dest> 
```

For example:

```
minikube mount ~/data:/data
```

### Load Balancer Access  

Use a tunnel instead of NodePort to access services externally:

```
minikube tunnel
```

This proxies services via localhost.

### Conclusion

Minikube is a valuable tool for developing locally against a real Kubernetes cluster. Consider using CI/CD pipelines to promote apps from minikube to permanent clusters.

## Chapter 7 - Conclusion

In this tutorial, we walked through installing, configuring, and using minikube to run a local Kubernetes cluster. Here are some key takeaways:

- Minikube is a handy tool for creating a Kubernetes environment on your workstation for development and testing.

- It allows you to validate app deployments and features like ingress, configmaps, RBAC, and more without needing a remote cluster.

- Different drivers like VirtualBox and VMware allow flexibility to match your OS and virtualization preferences. 

- Don't rely on minikube for production workloads. It is designed for experimentation on a single node.

- Consider using minikube to build CI/CD pipelines that promote code from local to dev, test, and finally production clusters.

- Addons extend minikube's functionality to add monitoring, logging, Istio integration, and other enterprise capabilities. 

- Best practices like caching images, mounting host folders, and using tunnels improve performance and productivity.

We covered a lot of ground on using minikube to build, deploy, and manage applications locally. Refer to the official minikube documentation for more information on configuration, troubleshooting, and limitations of a single node cluster.

I hope this tutorial provides a solid foundation for you to explore Kubernetes development further using minikube. Let me know if you have any other questions!
