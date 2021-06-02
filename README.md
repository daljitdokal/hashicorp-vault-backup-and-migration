
# Introduction
The purpose of this document is to create a new [HashiCorp Vault](https://www.hashicorp.com/) for secret management services. Secrets are defined as any form of sensitive credentials that need to be tightly controlled and monitored and can be used to unlock sensitive information. Secrets could be in the form of passwords, API keys, SSH keys, RSA tokens, or OTP.

## Unseal Vault Process
When a Vault server is started, it starts in a sealed state. In this state, Vault is configured to know where and how to access the physical storage, but doesn't know how to decrypt any of it.
Unsealing is the process of obtaining the plaintext master key necessary to read the decryption key to decrypt the data, allowing access to the Vault.

# Other Vaults
### Ansible Vault:
[Ansible Vault](https://docs.ansible.com/ansible/2.8/user_guide/vault.html#:~:text=Ansible%20Vault%20is%20a%20feature,or%20placed%20in%20source%20control.) is a feature of ansible that allows you to keep sensitive data such as passwords or keys in encrypted files, rather than as plaintext in playbooks or roles. These vault files can then be distributed or placed in source control.

It uses the ansible-vault command line utility tool for encrypting sensitive information using the AES256 algorithm. This provides symmetric encryption which is embedded to a defined password. A user can use the same password to either encrypt or decrypt files in order to access content.
