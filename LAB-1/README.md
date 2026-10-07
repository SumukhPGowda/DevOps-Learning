# Lab 1 – Kubernetes Getting Started

## Aim

To deploy an Nginx container as a Kubernetes Pod, expose it using a NodePort Service, and verify that the application is accessible.

## Environment

- Kubernetes cluster: 1-node Kubernetes environment
- Kubernetes version: v1.36.1
- kubectl: Kubernetes command-line tool
- Application image: `nginx`
- Pod name: `hello-k8s`
- Container port: `80`
- Service type: `NodePort`
- Verified NodePort during execution: `30365`

> Note: The lab was executed in a remote Kubernetes lab environment because the local 4 GB RAM machine could not satisfy Minikube's enforced 1800 MiB minimum Docker memory requirement. The Kubernetes commands and workload remain the same.

## Architecture

```text
Client
  |
  v
NodePort Service: hello-k8s
80:30365/TCP
  |
  v
Pod: hello-k8s
  |
  v
Nginx container
Port 80
```

## Procedure

### 1. Verify the Kubernetes node

```bash
kubectl get nodes
```

The node was reported as `Ready`.

### 2. Create the Nginx Pod

```bash
kubectl run hello-k8s --image=nginx --port=80
```

Output:

```text
pod/hello-k8s created
```

### 3. Verify the Pod

```bash
kubectl get pods
```

Observed result:

```text
hello-k8s   1/1   Running   0
```

### 4. Expose the Pod using a NodePort Service

```bash
kubectl expose pod hello-k8s --type=NodePort --port=80
```

Output:

```text
service/hello-k8s exposed
```

### 5. Verify the Service

```bash
kubectl get svc hello-k8s
```

Observed result:

```text
NAME        TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)
hello-k8s   NodePort   10.105.159.156  <none>        80:30365/TCP
```

The NodePort was dynamically assigned as `30365` during this execution.

### 6. Verify the Pod details

```bash
kubectl get pod hello-k8s -o wide
```

Observed result included:

```text
hello-k8s   1/1   Running   0   192.168.0.158   controlplane
```

### 7. Test the Nginx application

```bash
curl http://localhost:30365
```

The response contained:

```html
<title>Welcome to nginx!</title>
```

and:

```text
Welcome to nginx!
If you see this page, nginx is successfully installed and working.
```

### 8. Final verification

```bash
kubectl get all
```

Observed:

- Pod `hello-k8s`: `1/1 Running`
- Service `hello-k8s`: `NodePort`
- Service port: `80:30365/TCP`

## Kubernetes Manifests

The repository also contains equivalent declarative manifests:

- [`pod.yaml`](pod.yaml)
- [`service.yaml`](service.yaml)

These manifests are included to make the deployment reproducible. The actual execution during this lab used `kubectl run` and `kubectl expose`, as documented above.

## Evidence

### Pod and Service

![Kubernetes Pod and Service](screenshots/01-kubernetes-pod-service.png)

### Nginx Application Test

![Nginx curl test](screenshots/02-nginx-test.png)

### Final `kubectl get all`

![Final verification](screenshots/03-kubectl-get-all.png)

## Result

The Nginx application was successfully deployed as a Kubernetes Pod and exposed through a NodePort Service. The application was verified successfully using `curl`.



### 9. What was the NodePort in this execution?
`30365`.

### 10. What was the final result?
The Nginx default web page was successfully returned through the NodePort Service.
