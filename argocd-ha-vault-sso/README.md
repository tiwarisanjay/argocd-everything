# Install Vault 
Follow Document [Run Vault on Kubernetes](https://developer.hashicorp.com/vault/docs/platform/k8s/helm/run#how-to) or Follow following commands 
```bash
helm repo add hashicorp https://helm.releases.hashicorp.com
helm search repo hashicorp/vault
helm install vault hashicorp/vault -n vault --create-namespace
```
- View vault UI 
```bash 
kubectl port-forward vault-0 8200:8200 -n vault
```
- Initialize and unseal Vault
```
$kubectl get pods -l app.kubernetes.io/name=vault
NAME                                    READY   STATUS    RESTARTS   AGE
vault-0                                 0/1     Running   0          1m49s
```
- Initialize one Vault server with the default number of key shares and default key threshold:
```
$kubectl exec -ti vault-0 -n vault -- vault operator init
Unseal Key 1: MBFSDepD9E6whREc6Dj+k3pMaKJ6cCnCUWcySJQymObb
Unseal Key 2: zQj4v22k9ixegS+94HJwmIaWLBL3nZHe1i+b/wHz25fr
Unseal Key 3: 7dbPPeeGGW3SmeBFFo04peCKkXFuuyKc8b2DuntA4VU5
Unseal Key 4: tLt+ME7Z7hYUATfWnuQdfCEgnKA2L173dptAwfmenCdf
Unseal Key 5: vYt9bxLr0+OzJ8m7c7cNMFj7nvdLljj0xWRbpLezFAI9

Initial Root Token: s.zJNwZlRrqISjyBHFMiEca6GF
```
- The output displays the key shares and initial root key generated. Unseal the Vault server with the key shares until the key threshold is met:
```
## Unseal the first vault server until it reaches the key threshold
$ kubectl exec -ti vault-0 -n vault -- vault operator unseal # ... Unseal Key 1
$ kubectl exec -ti vault-0 -n vault -- vault operator unseal # ... Unseal Key 2
$ kubectl exec -ti vault-0 -n vault -- vault operator unseal # ... Unseal Key 3
```
- Command Line login. Use document [Login](https://developer.hashicorp.com/vault/docs/commands/login) for more info
```
$ vault login 
Token (will be hidden): 
WARNING! The VAULT_TOKEN environment variable is set! The value of this
variable will take precedence; if this is unwanted please unset VAULT_TOKEN or
update its value accordingly.

Success! You are now authenticated. The token information displayed below
is already stored in the token helper. You do NOT need to run "vault login"
again. Future Vault requests will automatically use this token.

Key                  Value
---                  -----
token                s.cdWzApasdkfjkasdqMHnuv
token_accessor       o03balkkjadskdfjd+uw4k
token_duration       ∞
token_renewable      false
token_policies       ["root"]
identity_policies    []
policies             ["root"]
```
- We will use this setup for vault connection for secret.
## APP ROLE 
- Login with your root token
- export vault address
    ```
    export VAULT_ADDR=http://127.0.0.1:8200
    ```
- Enable new kv engine
    ```
    vault secrets enable -version=2 -path=argocd kv
    ```
- To list all your current secerets 
    ```
    vault secrets list
    ```
- Create test secert
    ```
    vault kv put argocd/mysecret foo=bar
    ```
- Enable approle at path approle
    ```
    vault auth enable approle
    ```
- Create a vault policy 
    ```
    $vault policy write read-policy -<<EOF
    # Read-only permission on secrets stored at 'argocd/data'
    path "argocd/*" {
    capabilities = [ "read", "list" ]
    }
    EOF
    ```
- Now create a role 
    ```
    vault write auth/approle/role/argocd token_policies="read-policy"
    ```
- Read Policy 
    ```
    vault read auth/approle/role/argocd
    ```
- Get role-id 
    ```
    $vault read auth/approle/role/argocd/role-id

    Key     Value
    ---     -----
    role_id 675a50e7-cfe0-be76-e35f-49ec009731ea
    ```
- Get Secert ID 
    ```
     $vault write -force auth/approle/role/argocd/secret-id

    Key                 Value
    ---                 -----
    secret_id           ed0a642f-2acf-c2da-232f-1b21300d5f29
    secret_id_accessor  a240a31f-270a-4765-64bd-94ba1f65703c
    ```
### TEST if AppRole you cerate is working or not
- Login using your app role
    ```
    vault write auth/approle/login role_id="675a50e7-cfe0-be76-e35f-49ec009731ea" \
    secret_id="ed0a642f-2acf-c2da-232f-1b21300d5f29"
    ```
- List secret 
    ```
    vault kv list argocd/
    ```
- Read Secert 
    ```
    vault kv get test3/mysecret
    ```
# Install ArgoCD
