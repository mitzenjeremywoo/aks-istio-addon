export CLUSTER=my-istio-cluster
export RESOURCE_GROUP=istio-rg
export LOCATION=australiaeast

## Getting the service revision 
az aks mesh get-revisions --location australiaeast -o table


# enabling my cluster 
az aks mesh enable --resource-group istio-rg --name my-istio-cluster

# see if the cluster service mesh is enabled

az aks show --resource-group istio-rg --name my-istio-cluster  --query 'serviceMeshProfile.mode'


kubectl get pods -n aks-istio-system

## getting the istio revision currently in use
az aks show --resource-group istio-rg --name my-istio-cluster  --query 'serviceMeshProfile.istio.revisions'

## manual injections
kubectl apply -f <(istioctl kube-inject -f sample.yaml -i aks-istio-system -r asm-1-22) -n default

kubectl label namespace default istio.io/rev=asm-1-22


kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.18/samples/bookinfo/platform/kube/bookinfo.yaml


kubectl delete -f https://raw.githubusercontent.com/istio/istio/release-1.18/samples/bookinfo/platform/kube/bookinfo.yaml

## enabling external istio ingress

az aks show --name my-istio-cluster --resource-group istio-rg --query identity.type --output tsv

# Get the principal ID for a system-assigned managed identity.

CLIENT_ID=$(az aks show --name my-istio-cluster --resource-group istio-rg  --query identity.principalId --output tsv)

# Get the resource ID for the node resource group.
RG_SCOPE=$(az group show --name MC_istio-rg_my-istio-cluster_australiaeast --query id --output tsv)

# Assign the Network Contributor role to the managed identity,
# scoped to the node resource group.
az role assignment create --assignee ${CLIENT_ID} --role "Network Contributor" --scope ${RG_SCOPE}

az aks mesh enable-ingress-gateway --resource-group istio-rg --name my-istio-cluster --ingress-gateway-type external


## Apply gateway-external.yaml 

kubectl apply -f gateway-internal.yaml




## Enabling Internal Ingress

az aks mesh enable-ingress-gateway --resource-group $RESOURCE_GROUP --name $CLUSTER --ingress-gateway-type internal

## Apply gateway-internal.yaml 

kubectl apply -f gateway-internal.yaml

## Getting ready 

export INGRESS_HOST_INTERNAL=$(kubectl -n aks-istio-ingress get service aks-istio-ingressgateway-internal -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
export INGRESS_PORT_INTERNAL=$(kubectl -n aks-istio-ingress get service aks-istio-ingressgateway-internal -o jsonpath='{.spec.ports[?(@.name=="http2")].port}')
export GATEWAY_URL_INTERNAL=$INGRESS_HOST_INTERNAL:$INGRESS_PORT_INTERNAL