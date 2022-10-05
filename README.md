# ArgoCD with Istio and GKE 

## Pre-requisites

- This is for your production setup. So get the Istio/ASM(Anthos Service Mesh) running in your GKE. 
- Create a argocd namespace as following for ASM as following 
    ```yaml
    apiVersion: v1
    kind: Namespace
    metadata:
    labels:
        istio.io/rev: <asm version>
        kubernetes.io/metadata.name: argocd
    name: argocd
    ```
- Create argocd namespace as following for istio
    ```yaml
    apiVersion: v1
    kind: Namespace
    metadata:
    labels:
        istio-injection=enabled
        kubernetes.io/metadata.name: argocd
    name: argocd
    ```
- Get the ingress running with following label. # This label is used inside our gateway-vs.yaml Change it at line 7-8 if you have different name.
    ```
    istio: ingressgateway 
    ```
- Get a DNS for your ingress load balancer IP and change the DNS in gateway file with following command (Mac). Change manually for windoes:
```bash
dns=<your_dns>
sed -ie "s/argocd.example.com/${dns}/g" overlays/gateway-vs.yaml
```

## Install ArgoCD

- Now Install argocd with following command and it will start your argocd. 
```bash
cd argocd-istio-gke
kubectl apply -k . 
```
- As we have not setup login with sso it will be admin password. Use user as `admin`
- Password you can get from secret 
```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo 
```
- Login to ArgoCD server to upate password
```bash
argocd login <ARGOCD_SERVER> # Just  the IP:PORT or DNS:PORT
argocd account update-password
```

## For ArgoCD OIDC integration 
- Follow Instruction given at https://github.com/tiwarisanjay/argocd-dex for ArgoCD dex integration for any OIDC provider. 