# Vault-Kubernetes

## Steps


1. Enable the Key-Value Engine

```sh
vault secrets enable -version=2 -path="kv" kv
```

2.Adding the secrets to the vault
  ```sh

  vault kv put kv/dev/apps/service01 appkey="zsdkfjhj4534" apptoken="zsdasdfaskfjhj4534" 
  ```
3.Write the policy in the vault pod to read the secret 
  ```sh
  vault policy write demo-policy - <<EOH
  path "demo-app/*" {
    capabilities = ["read"]
  }
  EOH
  ```
4.Enable the kubernetes auth method and write the configs for the authentication
  ```sh
  vault auth enable kubernetes
  
  vault write auth/kubernetes/config token_reviewer_jwt="$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)" kubernetes_host="https://$KUBERNETES_PORT_443_TCP_ADDR:443" 
  kubernetes_ca_cert=@/var/run/secrets/kubernetes.io/serviceaccount/ca.crt
  ```
5.Connecting the sa with policy and create a role in vault
  ```sh
  kubectl create serviceaccount vault-auth

  vault write auth/kubernetes/role/webapp \
          bound_service_account_names=vault-auth \
          bound_service_account_namespaces=default \
          policies=demo-policy \
  ```
6.This annotation will trigger the webhook and the pod spec will change in runtime and add a init container and side-car container
   ```sh
   annotations:
      vault.hashicorp.com/agent-inject: 'true'
      vault.hashicorp.com/role: 'webapp'
      vault.hashicorp.com/agent-inject-secret-config.txt:
```
How the init container and the side-car conatiner is injected into the application pod
<img src="https://github.com/j-rin/Vault-Kubernetes/blob/main/Screenshot%20from%202024-06-28%2008-19-01.png" width="600" height="300">
How the secret is fetched from vault server
<img src="https://github.com/j-rin/Vault-Kubernetes/blob/main/Screenshot%20from%202024-06-28%2008-19-55.png" width="600" height="300">
Where is the secret stored for the application to access it
<img src="https://github.com/j-rin/Vault-Kubernetes/blob/main/Screenshot%20from%202024-06-28%2008-18-43.png" width="600" height="300">
