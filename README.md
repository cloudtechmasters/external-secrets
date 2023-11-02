# external-secrets

We are going to configure and use the External Secrets Operator. External Secrets Operator is a Kubernetes operator that integrates external secret management systems like AWS Secrets Manager, HashiCorp Vault, Google Secrets Manager, Azure Key Vault and many more. The operator reads information from external APIs and automatically injects the values into a Kubernetes Secret.

The goal of External Secrets Operator is to synchronize secrets from external APIs into Kubernetes. ESO is a collection of custom API resources - ExternalSecret, SecretStore and ClusterSecretStore that provide a user-friendly abstraction for the external API that stores and manages the lifecycle of the secrets for you.

Examples of secrets managers include HashiCorp Vault, AWS Secrets Manager, IBM Secrets Manager, Azure Key Vault, Akeyless, Google Secrets Manager, etc. To get secrets from you secrets manager into your cluster is by using the External Secrets Operator, a Kubernetes operator that enables you to integrate and read values from your external secrets management system and insert them as Secrets in your cluster.

Steps:
1. Install vault
2. Create secret in vault
3. Install External Secret Operator
4. Create SecretStore,ExternalSecret
5. Install Worpress site which is using MySQL DB (password for it stored in vault)


=================
1. Install vault

        sudo yum install -y yum-utils shadow-utils
        sudo yum-config-manager --add-repo https://rpm.releases.hashicorp.com/AmazonLinux/hashicorp.repo
        sudo yum -y install vault

Now start the vault server:

        sudo systemctl start vault

Access the Vault using : https://node_ip:8200

![image](https://github.com/cloudtechmasters/external-secrets/assets/68885738/15d2d210-5935-4618-996b-b60c848ae4a4)

Initialize it download the keys. Now open the UI again and provied keys to unseal vault and provide the root_token when it will ask for token.

Enable new secret engine:

![image](https://github.com/cloudtechmasters/external-secrets/assets/68885738/187365f8-a2e9-4638-93a4-30e022449688)

![image](https://github.com/cloudtechmasters/external-secrets/assets/68885738/d730b06d-ef58-4682-850d-c296a88de4c4)

now enable the engine with path sec

![Uploading image.png…]()


        
2. Create secret in vault
3. Install External Secret Operator
4. Create SecretStore,ExternalSecret
5. Install Worpress site which is using MySQL DB (password for it stored in vault)
