# Introduction
The purpose of this document is to create a tested migration approach for secrets between HashiCorp Vault instances.

# Migration Approach
We will be creating few testing environment with vault instances to mimic the current state and then create the end to end migration process to apply tested migration approach before starting the actual migration process.

## Prerequisites
* [Medusa](https://github.com/jonasvinther/medusa): Medusa is a cli tool currently for importing and exporting a json or yaml file into HashiCorp Vault and supports kv1 and kv2 Vault secret engines.

* [OC Playground (OpenShift 4.6)](https://learn.openshift.com/playgrounds/openshift46/): OpenShift Playgrounds give you a pre-configured environment ([OpenShift command-line interface (CLI)](https://docs.openshift.com/container-platform/4.2/cli_reference/openshift_cli/getting-started-cli.html) and the [Helm CLI](https://helm.sh/docs/helm/) installed) to start explore and experiment with an OpenShift cluster for 60 minutes. It is accessible from your browser without any downloads or configuration.

# Testing Migration Approach Process
* Got to [OC Playground (OpenShift 4.6)](https://learn.openshift.com/playgrounds/openshift46/) to start new playground instance.  
* Create a new project as "Old Vault" and Install the [HashiCorp Vault](https://learn.hashicorp.com/tutorials/vault/kubernetes-openshift?in=vault/kubernetes).
```bash
# Create new project
oc new-project oldvault --display-name 'Old Vault'

# Add the Hashicorp Helm repository.
helm repo add hashicorp https://helm.releases.hashicorp.com

# Update all the repositories to ensure helm is aware of the latest versions.
helm repo update

# Install the latest version of the Vault server running in development mode configured to work with OpenShift.
helm install vault hashicorp/vault \
    --set "global.openshift=true" \
    --set "server.dev.enabled=true"

# Display all the pods within the default namespace.
oc get pods

# The vault-0 pod runs a Vault server in development mode. The vault-agent-injector pod performs the injection based on the annotations present or patched on a deployment.
# Wait until the vault-0 pod and vault-agent-injector pod are running and ready (1/1).

# Create unseal key and root token
oc exec -it vault-0 -- /bin/sh
vault operator init -key-shares=1 -key-threshold=1

# Copy unseal key and root token
`Unseal Key 1: xxxxxxxx`
`Initial Root Token: yyyyyyyy`

# Unseal vault
vault operator unseal xxxxxxxx

exit
```
* Deployment secrets directly from Vault: Applications on pods can directly communicate with Vault to authenticate and request secrets. An application needs a service account, Vault secret and Vault policy to read the secret.
```bash
# Create the service account
touch service-account-webapp.yml
echo "apiVersion: v1" > service-account-webapp.yml 
echo "kind: ServiceAccount" > service-account-webapp.yml 
echo "metadata:" > service-account-webapp.yml 
echo "  name: webapp" > service-account-webapp.yml 

# Apply the service account.
oc apply --filename service-account-webapp.yml

# Start an interactive shell session on the vault-0 pod.
oc exec -it vault-0 -- /bin/sh

# Create the secret
vault kv put secret/webapp/config username="test-user" password="test-password"

# Define the read policy
vault policy write webapp - <<EOF
path "secret/data/webapp/config" {
  capabilities = ["read"]
}
EOF

exit
```

* Install [Medusa binary](https://github.com/jonasvinther/medusa) 

```bash
# Download Medusa binary
wget https://github.com/jonasvinther/medusa/releases/download/v0.2.2/medusa_0.2.2_linux_amd64.tar.gz

# Unzip the binary
tar xvf medusa_0.2.2_linux_amd64.tar.gz

# Validate if medusa works
./medusa --help
```

* Create the route to make vault accessible publicly.
  * Login in to OC and search for project "oldVault".
  * Create a new "route' to access the instance from externally.
  * Open the route URL and end the token as "root"

* Export secrets from "Old Vault" to .yaml file.

```bash
./medusa export secret --address="<OLD VAULT URL ADDRESS>" --token="xxxxxxxx" --format="yaml" --insecure > old_secrets.yaml
```

* Create a new project as "New Vault" and installed the [HashiCorp Vault](https://learn.hashicorp.com/tutorials/vault/kubernetes-openshift?in=vault/kubernetes)

```bash
# Create new project
oc new-project newvault --display-name 'New Vault'

# Add the Hashicorp Helm repository.
helm repo add hashicorp https://helm.releases.hashicorp.com

# Update all the repositories to ensure helm is aware of the latest versions.
helm repo update

# Install the latest version of the Vault server running in development mode configured to work with OpenShift.
helm install vault hashicorp/vault \
    --set "global.openshift=true" \
    --set "server.dev.enabled=true"

# Display all the pods within the default namespace.
oc get pods

# The vault-0 pod runs a Vault server in development mode. The vault-agent-injector pod performs the injection based on the annotations present or patched on a deployment.
# Wait until the vault-0 pod and vault-agent-injector pod are running and ready (1/1).

# Create unseal key and root token
oc exec -it vault-0 -- /bin/sh
vault operator init -key-shares=1 -key-threshold=1

# Copy unseal key and root token
`Unseal Key 1: xxxxxxxx`
`Initial Root Token: yyyyyyyy`

# Unseal vault
vault operator unseal xxxxxxxx

```
* Create the route to make vault accessible publicly.
  * Login in to OC and search for project "newVault".
  * Create a new "route' to access the instance from externally.
  * Open the route URL and end the token as "root"

* Import secrets from .yaml file to "New Vault" 

```bash
./medusa import secret old_secrets.yaml --address="<NEW VAULT URL ADDRESS>" --token="xxxxxxxx" --insecure
```
