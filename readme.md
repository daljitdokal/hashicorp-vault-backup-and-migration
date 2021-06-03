
# Introduction
The purpose of this document is to create a new [HashiCorp Vault](https://www.hashicorp.com/) for secret management services. Secrets are defined as any form of sensitive credentials that need to be tightly controlled and monitored and can be used to unlock sensitive information. Secrets could be in the form of passwords, API keys, SSH keys, RSA tokens, or OTP.

# Prerequisites
New secret manager instance is required to setup before we start the migration process. We will be using official HashiCorp Helm chart for installing and configuring Vault on Kubernetes. To use the charts, Helm must be configured for your Kubernetes cluster (Setting up Kubernetes and Helm is outside the scope)

The versions required are:

 - **Helm 3.0+**: This is the earliest version of Helm tested. It is possible it works with earlier versions but this chart is untested for those versions.
 - **Kubernetes 1.14+**: This is the earliest version of Kubernetes tested. It is possible that this chart works with earlier versions but it is untested.

# New Vault with Helm Chart
To install the latest version of this chart, add the [Hashicorp helm repository](https://github.com/daljitdokal/vault-helm) and run `helm install`:
```bash
$ helm repo add hashicorp https://helm.releases.hashicorp.com
"hashicorp" has been added to your repositories

$ helm install vault hashicorp/vault
```
Please see the many options supported in the values.yaml file. These are also fully documented directly on the [Vault website](https://www.vaultproject.io/docs/platform/k8s/helm) along with more detailed installation instructions.

## Unseal Vault Process
When a Vault server is started, it starts in a sealed state. In this state, Vault is configured to know where and how to access the physical storage, but doesn't know how to decrypt any of it.
Unsealing is the process of obtaining the plaintext master key necessary to read the decryption key to decrypt the data, allowing access to the Vault.

```bash
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

# Other Vaults
### Ansible Vault:
[Ansible Vault](https://docs.ansible.com/ansible/2.8/user_guide/vault.html#:~:text=Ansible%20Vault%20is%20a%20feature,or%20placed%20in%20source%20control.) is a feature of ansible that allows you to keep sensitive data such as passwords or keys in encrypted files, rather than as plaintext in playbooks or roles. These vault files can then be distributed or placed in source control.

It uses the ansible-vault command line utility tool for encrypting sensitive information using the AES256 algorithm. This provides symmetric encryption which is embedded to a defined password. A user can use the same password to either encrypt or decrypt files in order to access content.
