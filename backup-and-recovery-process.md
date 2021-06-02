
# Introduction
The purpose of this document is to configure backup and recovery process solution for new created HashiCorp vault.

## Prerequisites
* [Medusa](https://github.com/jonasvinther/medusa): Medusa is a cli tool currently for importing and exporting a json or yaml file into HashiCorp Vault and supports kv1 and kv2 Vault secret engines.
* [The RSA key-pair](https://en.wikipedia.org/wiki/RSA_(cryptosystem)): When exporting your Vault secrets using Medusa, the secrets are encrypted using the AES symmetric encryption algorithm. The 256-bit AES encryption key is randomly generated by Medusa every time the export command is being called. This ensures that both the exported secrets and AES enctyption key can be transfered safely between Vault instances.
The exported secrets and AES enctyption key can only be decrypted by a person who is in possession of the RSA private key.
```bash
# The RSA key-pair can be generated by the following two commands:
# Generate private key
openssl genrsa -out vault-private-key.pem 4096

# Generate public key
openssl rsa -in vault-private-key.pem -pubout -out vault-public-key.pem
```
* Manually creating `kv` secrets engine in new vault (same as old vault).


# Exporting vault secrets
Vault export is easy using Medusa. Simply use following command to export secrets to create backup with `date` from legacy vault. At this stage this script will be executed manually but in future this script will be used in CI/CD process to create daily backups.
```bash
#!/bin/sh

# Create variables
vaultAddress="https://<vault-address>";
vaultToken="xxxxxxxxxxxx";
secretsEngines=('common' 'dev' 'test' 'staging' 'prod');
getDate="$(date '+%Y%m%d')";
pubicKeyPath="./keys/vault-public-key.pem";
outputFolderPath="./backup";
encrypt="false"

# Try catch
{ 
    # Create folder for each date backup
    if [ -d "${outputFolderPath}/${getDate}" ]; then
        echo "Folder already exits."
    else
        mkdir "${outputFolderPath}/${getDate}";
    fi

    echo "Exporting started."
    # start exporting process
    for kv in "${secretsEngines[@]}";
    do
        ./medusa export ${kv} --address="${vaultAddress}" --token="${vaultToken}" --insecure --encrypt="${encrypt}" --public-key="${pubicKeyPath}" --output="${outputFolderPath}/${getDate}/${kv}.txt"
        echo " - Secrest engine '${kv}': completed."
    done
    echo "Exporting completed successfully!"
} || {
    echo "Error in $__EXCEPTION_SOURCE__ at line: $__EXCEPTION_LINE__!"
}
```

# Importing vault secrets
Please use the following commands to import the secrets to new vault.
```bash
#!/bin/sh

# Create variables
vaultAddress="https://<vault-address>;
vaultToken="xxxxxxxxxx";
secretsEngines=('common' 'dev' 'test' 'staging' 'prod');
getDate="$(date '+%Y%m%d')";
privateKeyPath="./keys/vault-private-key.pem";
outputFolderPath="./backup";
decrypt="false"

# Try catch
{ 
    # Create folder for each date backup
    if [ -d "${outputFolderPath}/${getDate}" ]; then
        echo "Importing started."
        # start exporting process
        for kv in "${secretsEngines[@]}";
        do
            ./medusa import ${kv} "${outputFolderPath}/${getDate}/${kv}.txt" --address="${vaultAddress}" --token="${vaultToken}" --insecure --decrypt="${decrypt}" --private-key="${privateKeyPath}"
            echo " - Secrest engine '${kv}' completed."
            # sleep 0.5 # Waits 0.5 second.
        done
        echo "Importing completed successfully!"
    else
        echo "Invalid folder path '${outputFolderPath}/${getDate}'"
    fi
} || {
    echo "Error in $__EXCEPTION_SOURCE__ at line: $__EXCEPTION_LINE__!"
}

```


