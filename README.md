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
   

        cp /usr/bin/vault /usr/local/sbin
        mkdir /etc/vault
        mkdir -p /var/lib/vault/data
        useradd --system --home /etc/vault --shell /bin/false vault
        chown -R vault:vault /etc/vault /var/lib/vault/

Copy vault.service and config.hcl from the vault directory to the below paths respectively: /etc/systemd/system/vault.service,/etc/vault/config.hcl 

        sudo systemctl daemon-reload
        sudo systemctl enable --now vault

Created symlink from /etc/systemd/system/multi-user.target.wants/vault.service to /etc/systemd/system/vault.service.

        systemctl status vault

        export VAULT_ADDR=http://127.0.0.1:8200
        vault operator init |tee  /etc/vault/init.file  

        # cat /etc/vault/init.file
        Unseal Key 1: 8w/vmDxrxTPovyxHz8e0wfixH4IUb08JozU/hvahZpV7
        Unseal Key 2: P8TVjb2jUvqwp8d3F306vlEHlJ4LGbfRo4BfFvhNc5xb
        Unseal Key 3: xgkP1Yh4hprcgRMpscVEkMAT0q+nOQlFimU2B6uWCEkJ
        Unseal Key 4: 1nn1rbAh+2VrEMbLWd2RLsHItSG5tK1STsJDHXQ8dJnN
        Unseal Key 5: yZlRakRA1Caxs7HcP7Kq5uM6gP/vcRfdyB8qHl14zRy0
        
        Initial Root Token: s.UQwEqH4ZevMplOBGjdmQo4eS
        
        Vault initialized with 5 key shares and a key threshold of 3. Please securely
        distribute the key shares printed above. When the Vault is re-sealed,
        restarted, or stopped, you must supply at least 3 of these keys to unseal it
        before it can start servicing requests.
        
        Vault does not store the generated master key. Without at least 3 key to
        reconstruct the master key, Vault will remain permanently sealed!
        
        It is possible to generate new unseal keys, provided you have a quorum of
        existing unseal keys shares. See "vault operator rekey" for more information.



        Access the Vault using : http://node_ip:8200

![image](https://github.com/cloudtechmasters/external-secrets/assets/68885738/15d2d210-5935-4618-996b-b60c848ae4a4)

Initialize it download the keys. Now open the UI again and provied keys to unseal vault and provide the root_token when it will ask for token.

Enable new secret engine:

![image](https://github.com/cloudtechmasters/external-secrets/assets/68885738/187365f8-a2e9-4638-93a4-30e022449688)

![image](https://github.com/cloudtechmasters/external-secrets/assets/68885738/d730b06d-ef58-4682-850d-c296a88de4c4)

now enable the engine with path sec

![image](https://github.com/cloudtechmasters/external-secrets/assets/68885738/1e479ec8-d5fe-4200-9d24-4f666e91108a)

        
2. Create secret in vault

![image](https://github.com/cloudtechmasters/external-secrets/assets/68885738/38491db0-390e-43c0-ae89-600677132c11)

3. Install External Secret Operator

        helm repo add external-secrets https://charts.external-secrets.io
        
        helm install external-secrets \
            external-secrets/external-secrets \
            -n tools \
            --create-namespace \
            --set installCRDs=true

        kubectl get pods -n tools

        NAME                                                READY   STATUS    RESTARTS   AGE
        external-secrets-969fb45c8-9n8kk                    1/1     Running   0          22h
        external-secrets-cert-controller-7f99594b48-dr4wl   1/1     Running   0          22h
        external-secrets-webhook-75878d4b89-mgtwq           1/1     Running   0          22h


5. Create SecretStore,ExternalSecret

First, create a SecretStore with a vault backend. For the sake of simplicity we'll use a static token root:

        apiVersion: external-secrets.io/v1beta1
        kind: SecretStore
        metadata:
          name: vault-backend
        spec:
          provider:
            vault:
              server: "http://my.vault.server:8200"
              path: "secret"
              version: "v1"
              auth:
                # points to a secret that contains a vault token
                # https://www.vaultproject.io/docs/auth/token
                tokenSecretRef:
                  name: "vault-token"
                  key: "token"
        ---
        apiVersion: v1
        kind: Secret
        metadata:
          name: vault-token
        data:
          token: cm9vdA== # "root"

   
6. Install Worpress site which is using MySQL DB (password for it stored in vault)
