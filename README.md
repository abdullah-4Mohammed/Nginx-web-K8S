# k8s-nginx:

 if you're using hostPath to create a PersistentVolume (PV) in Kubernetes, you need to ensure that the specified path (/mnt/data in your example) exists on the node where the PV will be mounted.

In this case, the hostPath field is used to create a PV on the node's file system at /mnt/data. The nginx container in your Deployment manifest will then mount this PV at /usr/share/nginx/html, which is the default document root for the nginx web server. So, you will also need to create an HTML file at /mnt/data on the node's file system, which will be served by the nginx web server.

If the specified path does not exist on the node, Kubernetes will fail to create the PV, and your Pod or Deployment will fail to start. So, you should ensure that the path exists and that the appropriate permissions are set before creating the PV.

Note that using hostPath for PersistentVolumes can have security implications, especially if the path being used is on the node's file system. So, you should be careful when using hostPath and ensure that appropriate security measures are in place.

To create the /mnt/data path and an HTML file inside it, you can use the following steps:

SSH into the minikube VM using the following command:
minikube ssh
Inside the VM, navigate to the /mnt/data directory using the following command:
cd /mnt/data
Create an HTML file in the /mnt/data directory using your favorite text editor. For example, you can create a file called index.html with the following content:
<html>
<body>
<h1>Hello, world!</h1>
</body>
</html>

Save the index.html file and exit the text editor.

Now, when you create a PersistentVolumeClaim (PVC) and a PersistentVolume (PV) with hostPath using /mnt/data as the path, the HTML file you created should be accessible inside the Pod's container at /usr/share/nginx/html.
the nano editor is not installed in the minikube VM. You can install it using the following command:
sudo apt-get update
sudo apt-get install nano
You should now be able to create the index.html file using the nano command:
sudo nano /mnt/data/index.html
This will open the nano editor with a blank file. You can then type or paste the HTML code for your web page into the file, save it, and exit the editor.

deployment files
In the "selector" section, the "matchLabels" field specifies the label 
selector that the Deployment uses to select the pods it will manage.
 The label selector matches against the labels on all the pods in
  the Kubernetes cluster, not just the pods created by 
  the Deployment YAML file. If a pod has labels that match 
  the "matchLabels" field of the Deployment's selector, 
  then the Deployment will manage that pod.

  In the "template" section, the YAML file specifies the pod template 
  that the Deployment uses to create new pods. The label selectors 
  in the Deployment's "selector" section and the labels in the 
  pod template's "metadata" section work together to ensure that 
  the Deployment manages the pods it creates.

Simple NginX and curl deployment to minikube

The YaML files provided in the [manifests](manifests) folder can be used to
deploy a simple `NginX` stack to a given Kubernetes cluster, with a pod
providing `curl`. They are intended to be used for demonstration and training
purposes.

## Requirements

## Content


## Installation

Clone this repository:

Make sure you have configured `minikube` as current context:

```bash
kubectl config use-context minikube
```

To use these manifests, run:

```bash
kubectl apply -R -f manifests
```

## Usage

To view only the `nginx` pods, run:

```bash
kubectl get pods -l app=nginx
```

To view only the `curl` pod, run:

```bash
kubectl get pods -l app=curl
```

To identify a `nginx` pod for shell usage, run:

```bash
NGINX_POD=$(kubectl get pod -l app=nginx -o jsonpath="{.items[0].metadata.name}")
```

To identify the `curl` pod for shell usage, run:

```bash
CURL_POD=$(kubectl get pod -l app=curl -o jsonpath="{.items[0].metadata.name}")
```

To run a shell on the `nginx` pod, run:

```bash
kubectl exec ${NGINX_POD} --tty -i -- bash
```

To get a shell on the `curl` pod, run:

```bash
kubectl exec ${CURL_POD} --tty -i -- sh
```

You can also run commands in the `curl` pod directly, as follows:

```bash
kubectl exec ${CURL_POD} -- nslookup nginx
kubectl exec ${CURL_POD} -- nslookup nginx.default.svc.cluster.local

kubectl exec ${CURL_POD} -- curl -v -s http://nginx/
kubectl exec ${CURL_POD} -- curl -v -s http://nginx.default.svc.cluster.local/
```

## Scaling

To scale the deployment up or down, run (i.e. to scale to 3 pods):

```bash
kubectl scale deployment nginx --replicas=3
```

To disable keepalives (and show how the service distributes the requests across
the different pods running) while executing `curl`, add the HTTP request header
`"Connection: close"`:

```bash
kubectl exec ${CURL_POD} -- curl -v -s -H "Connection: close" http://nginx/
```

Search for the `X-Kubernetes-Pod` header in the response of the HTTP request.
You will see that it changes to the names of the `nginx` pods running:

```bash
kubectl get pods -l app=nginx
```

## Browser access

To view the service in a web browser, get the URL of the `nginx` service from
`minikube`:

```bash
minikube service nginx --url
```

Open the resulting URL in your favorite browser. Also use the browser's
developer tools to see the HTTP requests and responses in the browser.

## Uninstallion

To uninstall all objects, run:

```bash
kubectl delete -R -f manifests
```
