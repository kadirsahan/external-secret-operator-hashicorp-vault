
# helm installation on the master node
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 
get_helm.sh 

# install K9s ubuntu
curl -sS https://webi.sh/k9s | sh # this is better , snap has an issue.
export KUBECONFIG=$HOME/.kube/config
source ~/.config/envman/PATH.env

# Install Vault
helm repo add hashicorp https://helm.releases.hashicorp.com
helm install vault hashicorp/vault -n vault --create-namespace

# unseal the vault 
kubectl exec -ti vault-0 -n vault -- vault operator init

Unseal Key 1: c7ygURyqgiZ6H0LXUngyZw7Ct786e5JlbkkippVVlaHq
Unseal Key 2: 324ul+TqFrPitCceQ0lTN8H+WDd3OZal99fEhN0HFIAK
Unseal Key 3: ynIScHAVWcGYOE6nxtpdTD7wBuO8/0nk0qNRqjL0+nxm
Unseal Key 4: a2TRHRTI5ioDjudvOHNvKNwkKWECygqBKYaCvbg/YJoy
Unseal Key 5: 2pcvoMyuXmzu+5NTLVxkRhbPqYwsHHtFnTJnmw9U5cpG

Initial Root Token: hvs.f9wUXU0gEMfo1AJLODgMywW2

kubectl exec -ti vault-0 -n vault -- vault operator unseal

# login to vault-0 container
export VAULT_ADDR=http://127.0.0.1:8200
vault login

# Enable KV secret
vault secrets enable -version=1 -path=postgres kv

vault kv put postgres/secret userpg=passwdpg

# Enable AppRole
vault auth enable approle

# Create a read policy
vault policy write read-policy -<<EOF
path "postgres/*" {
capabilities = [ "read", "list" ]
}
EOF

# Create a role to bind APPROLE(postgres) with read policy (read-policy)

vault write auth/approle/role/postgres token_policies="read-policy"

vault read auth/approle/role/postgres

# get the RoleID
vault read auth/approle/role/postgres/role-id

/ $ vault read auth/approle/role/postgres/role-id
Key        Value
---        -----
role_id    5bd8060e-d284-807f-5210-09e3687ebb02

# get your secret_id

vault write -force auth/approle/role/postgres/secret-id

/ $ vault write -force auth/approle/role/postgres/secret-id
Key                   Value
---                   -----
secret_id             75a4eac5-13a6-8555-8115-04af45f5f45e
secret_id_accessor    b287c79d-7a27-0b8b-6260-31ab09fa46b3
secret_id_num_uses    0
secret_id_ttl         0s

