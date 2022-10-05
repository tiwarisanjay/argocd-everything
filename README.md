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