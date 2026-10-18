# Exercise 10 — Multi-Node Minikube with Multiple Applications

## Aim
Deploy two Flask applications to a three-node Minikube cluster and distribute replicas across nodes:
- **Product Catalog:** 2 replicas, `GET /products`
- **Shopping Cart:** 3 replicas, `GET /cart` and `POST /cart`
- Both apps are exposed with Kubernetes NodePort Services.
- Required pod anti-affinity spreads replicas of each app across different Kubernetes nodes.

> **Resource note:** This lab needs Docker Desktop and Minikube to create three Kubernetes nodes. The available Docker memory on some 4 GB systems may be insufficient. Do not delete the existing `minikube` profile; use the separate `devops-multinode` profile below.

## Files
- `product_catalog.py`, `shopping_cart.py` — Flask apps
- `Dockerfile.product`, `Dockerfile.shopping` — container build files
- `namespace.yaml` — creates `devops-exercise`
- `*_deployment.yaml` — deployments with replica counts, probes, resource requests/limits, and pod anti-affinity
- `*_service.yaml` — NodePort services

## 1. Start the three-node cluster
Open PowerShell after Docker Desktop reports that it is running:

```powershell
docker version
minikube start --nodes 3 -p devops-multinode --driver=docker --force
kubectl --context=devops-multinode get nodes -o wide
```

The final command should show three nodes in `Ready` state. The `--force` flag is included to accommodate the lab's Docker-driver/root-privilege warning in the source instructions.

If startup times out, inspect `docker version`, `docker ps -a`, and `minikube profile list` before retrying. On a low-memory machine, the three-node cluster may not fit. Do not remove the existing `minikube` profile to make room without first checking what it contains.

## 2. Build the two images
Run these commands from the `Exercise-10` directory:

```powershell
docker build -t product-catalog:latest -f Dockerfile.product .
docker build -t shopping-cart:latest -f Dockerfile.shopping .
```

Load the images into the Minikube profile so pods can use them without pulling from a remote registry:

```powershell
minikube -p devops-multinode image load product-catalog:latest
minikube -p devops-multinode image load shopping-cart:latest
```

## 3. Deploy namespace, apps, and services
Set the kubectl context explicitly to avoid applying resources to the wrong cluster:

```powershell
kubectl config use-context devops-multinode
kubectl apply -f namespace.yaml
kubectl apply -f product_catalog_deployment.yaml
kubectl apply -f shopping_cart_deployment.yaml
kubectl apply -f product_catalog_service.yaml
kubectl apply -f shopping_cart_service.yaml
```

Wait for both deployments:

```powershell
kubectl -n devops-exercise rollout status deployment/product-catalog
kubectl -n devops-exercise rollout status deployment/shopping-cart
```

## 4. Verify replicas and node distribution

```powershell
kubectl -n devops-exercise get deployments,replicasets,pods -o wide
kubectl -n devops-exercise get services
```

Expected: Product Catalog has 2/2 ready replicas; Shopping Cart has 3/3 ready replicas. Each application's replicas should be on different nodes because of required pod anti-affinity. If there are fewer than three Ready nodes, the three Shopping Cart replicas will remain Pending by design.

## 5. Access and test the services
In separate PowerShell windows, start service tunnels and leave each window open:

```powershell
minikube -p devops-multinode -n devops-exercise service product-catalog-service --url
```

```powershell
minikube -p devops-multinode -n devops-exercise service shopping-cart-service --url
```

Use the actual URLs printed by those commands. In another PowerShell window, set the URL variables to the printed addresses:

```powershell
$productUrl = "PASTE_PRODUCT_CATALOG_URL_HERE"
$cartUrl = "PASTE_SHOPPING_CART_URL_HERE"

Invoke-RestMethod "$productUrl/products"
Invoke-RestMethod "$cartUrl/cart"
Invoke-RestMethod -Method Post -Uri "$cartUrl/cart" -ContentType "application/json" -Body '{"id":1,"name":"Laptop","quantity":1}'
Invoke-RestMethod "$cartUrl/cart"
```

Expected behavior:
- `/products` returns Laptop, Phone, and Headphones.
- A fresh `/cart` returns `[]`.
- Posting a cart item returns HTTP 201 and the item.
- A subsequent `/cart` shows the added item. Cart state is in-memory and is not shared between replicas; persistence/load-balanced session behavior is outside this lab's scope.

## 6. Useful troubleshooting commands

```powershell
kubectl --context=devops-multinode get nodes
kubectl -n devops-exercise get pods -o wide
kubectl -n devops-exercise describe pod POD_NAME
kubectl -n devops-exercise logs deployment/product-catalog
kubectl -n devops-exercise logs deployment/shopping-cart
```

## Conclusion
This exercise demonstrates multi-node Kubernetes scheduling, Deployments and their ReplicaSets, pod anti-affinity for replica distribution, NodePort Services, and basic HTTP testing of two containerized Flask applications.
