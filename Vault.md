# Vault Demo

## Install vault on local machine (Mac)
```bash
brew tap hashicorp/tap
brew install hashicorp/tap/vault
```

## Run vault on dev mode
```bash
vault server -dev
vault server -dev -dev-root-token-id=root
```
Modes:
> Dev
> Standalone
> HA

## Set Vault Address and root token
Vault Cli -> Vault Server
```bash
export VAULT_ADDR="http://127.0.0.1:8200"
export VAULT_ROOT="s.V9gSXzMZ7qni5qTaYpDfTMg2"
```

## Check vault status

```bash
vault status
```

## Play with UI
- Go to Vault Addr on browser on Login with root key

## Secret engines

- Add new KV
- Revisions
    - Create new revision
    - Check existing revisions


## Add KV
```bash
vault kv put secret/my-application apikey="token" passkey="mypassword"
```

## Retrieve kv
```bash
vault kv get -format=json -version=3 secret/my-application
```

## Create new tokens
```bash
vault token create
```

## Login with token
```bash
vault login <TOKEN>
```

## Create kv in the name of restricted
```bash
vault kv put secret/restricted apikey="xxx"
```

## Create policy
vault policy write restricted-read - <<EOF
```hcl
path "secret/data/restricted" {
    capabilities = ["read"]
}
```
EOF

## Attach policy to new token
```bash
vault token create -policy=restricted-read
```

## Read secret
```bash
vault kv get secret/restricted
```

## Write secret
```bash
vault kv put secret/restricted apikey="xxx"
```

## Seal Vault
```bash
vault operator seal
```

## Unseal Vault
```bash
vault operator unseal
```
