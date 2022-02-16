# Create k8s cluster

kind create cluster --name forcertmanager --image kindest/node:v1.21.1 --config kind.yaml

# NGINX Ingress controller

https://kubernetes.github.io/ingress-nginx/deploy/#quick-start

helm upgrade --install ingress-nginx ingress-nginx \
  --repo https://kubernetes.github.io/ingress-nginx \
  --namespace ingress-nginx --create-namespace --values nginx-values.yaml

# Cert manager

https://cert-manager.io/docs/installation/helm/

helm install \
  cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --version v1.7.1 \
  --set installCRDs=true

kubectl get pods -ncert-manager -ojsonpath="{range .items[*]}{.spec.containers[*].image}{\"\n\"}{end}"
quay.io/jetstack/cert-manager-controller:v1.7.1
quay.io/jetstack/cert-manager-cainjector:v1.7.1
quay.io/jetstack/cert-manager-webhook:v1.7.1


# Setup ca certificate

openssl genrsa -out ca.key 2048
openssl req -x509 -new -nodes -key ca.key -subj "/CN=Bence CA" -days 3650 -reqexts v3_req -extensions v3_ca -out ca.crt
openssl x509 -noout -text -in ca.crt

kubectl create secret tls bence-ca-crt --cert="./ca.crt" --key="./ca.key"

# Create CA Issuer

kubectl create -f issuer.yaml

# Example application

kubectl apply -f application.yaml -f ingress.yaml

import ca.crt to browser

Go to  https://www.mydomain.com:32443

# Cleanup

kind delete cluster --name forcertmanager