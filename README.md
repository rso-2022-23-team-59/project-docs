# General information about the project

## Development

### First start

To start working you need a suitable development structure where all repositories are in the same directory

such as:

rso/
    project-docs/
    product-catalog/
    store-catalog/
    ...


Then you can start all microservices with

```bash
docker compose up
```

### Useful commands:

Run only one service
```bash
docker compose up product-catalog
```

Build only one service
```bash
docker compose up product-catalog --build
```

Enter container bash
```bash
 docker compose exec -it product-catalog bash
 ```

## Running in production

First, install [Azure CLI](https://learn.microsoft.com/sl-si/cli/azure/install-azure-cli?view=azure-cli-latest) and connect to Azure Kubernetes service.

```bash
# Set Azure subscription id
az account set --subscription <SUBSCRIPTION_ID>

# Connect to Azure Kubernetes service in resource group PriceComparisonRG
az aks get-credentials --resource-group PriceComparisonRG --name price-comparison
```

Then, start the [nginx ingress controller](https://kubernetes.github.io/ingress-nginx/deploy/#quick-start) responsible for redirecting requests to corresponding microservices.

```bash
# Start nginx Ingress Controller
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.5.1/deploy/static/provider/cloud/deploy.yaml

# Wait until the nginx Ingress Controller starts
kubectl wait --namespace ingress-nginx --for=condition=ready pod --selector=app.kubernetes.io/component=controller --timeout=120s
```

Add a Kubernetes secret with database password.

```bash
# Add a secret to Kubernetes (required by product-catalog and shopping-cart Docker images)
# For more information see https://kubernetes.io/docs/tasks/configmap-secret/managing-secret-using-kubectl/
kubectl create secret generic product-catalog-database-password --from-literal=KUMULUZEE_DATASOURCES0_PASSWORD='<PASSWORD>'
kubectl create secret generic store-catalog-database-password --from-literal=KUMULUZEE_DATASOURCES0_PASSWORD='<PASSWORD>'
kubectl create secret generic shopping-cart-database-password --from-literal=KUMULUZEE_DATASOURCES0_PASSWORD='<PASSWORD>'
kubectl create secret generic notification-catalog-database-password --from-literal=KUMULUZEE_DATASOURCES0_PASSWORD='<PASSWORD>'
kubectl create secret generic rabbitmq-password --from-literal=RABBITMQ_DEFAULT_PASS='<PASSWORD>'

```

Start the ingress and display its public IP.

```bash
# Create an Ingress for routing data to microservices
kubectl apply -f ingress.yaml

# Show IP address of created Ingress Controller
kubectl get service ingress-nginx-controller --namespace=ingress-nginx
```

The Ingress defines the following endpoints:

| Endpoint | Microservice | Example URL |
| -------- | ------------ | ----------- |
| `/frontend` | `frontend:8080` | `http://<EXTERNAL_IP>/frontend` |
| `/product-catalog` | `product-catalog:8080` | `http://<EXTERNAL_IP>/product-catalog/v1/products` |¸
| `/store-catalog` | `store-catalog:8080` | `http://<EXTERNAL_IP>/store-catalog/v1/stores` |
| `/shopping-cart` | `shopping-cart:8080` | `http://<EXTERNAL_IP>/shopping-cart/v1/shopping-carts/1` |

### Create Kubernetes deployments and services

To create Kubernetes deployments and services, run the following commands.

```bash
kubectl apply -f rabbitmq.yaml
kubectl apply -f consul.yaml
kubectl apply -f frontend/k8s/deployment.yaml
kubectl apply -f product-catalog/k8s/deployment.yaml
kubectl apply -f shopping-cart/k8s/deployment.yaml
kubectl apply -f store-catalog/k8s/deployment.yaml
```