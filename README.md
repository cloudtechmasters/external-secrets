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
**1. Install vault**

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
        
        Initial Root Token: hvs.unbdbSsB0m6CR3980Fyt7FC3
        
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

# Enable new secret engine:

![image](https://github.com/cloudtechmasters/external-secrets/assets/68885738/6b54867d-38d0-4ead-86a9-8268184025ae)

now enable the engine with path secret:

![image](https://github.com/cloudtechmasters/external-secrets/assets/68885738/71556ccc-914f-4428-bc1a-9a8c039a03a8)


**2. Create secret in vault**

![image](https://github.com/cloudtechmasters/external-secrets/assets/68885738/16de486f-6566-4257-bd3c-e14e2a84f295)

![image](https://github.com/cloudtechmasters/external-secrets/assets/68885738/0f4046d6-c8cf-4333-b7e4-3ae1ea1c98a6)


**3. Install External Secret Operator**

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


**5. Create SecretStore,ExternalSecret**

# Create vault-token for the secretstore to connect with vault

 kubectl create secret generic vault-token --from-literal=token=root_token
 
         kubectl create secret generic vault-token --from-literal=token=hvs.unbdbSsB0m6CR3980Fyt7FC3

First, create a SecretStore with a vault backend. 
        
        apiVersion: external-secrets.io/v1beta1
        kind: SecretStore
        metadata:
          name: vault-backend
        spec:
          provider:
            vault:
              server: "http://my.vault.server:8200"
              path: "secret"
              version: "v2"
              auth:
                # points to a secret that contains a vault token
                # https://www.vaultproject.io/docs/auth/token
                tokenSecretRef:
                  name: "vault-token"
                  key: "token"


        $ kubectl apply -f secret.yaml
        secretstore.external-secrets.io/vault-backend created

        $ kubectl get secretstore
        NAME            AGE   STATUS   CAPABILITIES   READY
        vault-backend   9s    Valid    ReadWrite      True

# Now create a ExternalSecret that uses the above SecretStore:

        apiVersion: external-secrets.io/v1beta1
        kind: ExternalSecret
        metadata:
          name: vault-example
        spec:
          refreshInterval: "15s"
          secretStoreRef:
            name: vault-backend
            kind: SecretStore
          target:
            name: mysql-pass  -- Secret will be created with this name
          data:
          - secretKey: password  -- Secret will be created with this key
            remoteRef:
              key: secret/sec  -- path to the secret : secret/sec
              property: password  -- key for secret (passord=admin)


        kubectl apply -f externalSecret.yaml
        externalsecret.external-secrets.io/vault-example configured

This will create externalsecret vault-example
        
        $ kubectl get externalsecret
        NAME            STORE           REFRESH INTERVAL   STATUS         READY
        vault-example   vault-backend   15s                SecretSynced   True

This will create the below secret mysql-pass, with the key password
        
        $ k get secret
        NAME          TYPE     DATA   AGE
        mysql-pass    Opaque   1      5m13s
        vault-token   Opaque   1      48m

**6. Install Worpress site which is using MySQL DB (password for it stored in vault)**

        kubectl apply -f mysql-deployment.yaml
        service/wordpress-mysql created
        persistentvolumeclaim/mysql-pv-claim created
        deployment.apps/wordpress-mysql created

kubectl get pods
NAME                              READY   STATUS    RESTARTS   AGE
wordpress-mysql-b759dbb45-2j89n   1/1     Running   0          2m17s

To verify if user WordPress is created using the password we created in the vault i.e admin use below:

        kubectl exec -it wordpress_pod_name -- mysql -u wordpress -p -e "show databases"

It will then prompt for the password/

        $ kubectl exec -it wordpress-mysql-b759dbb45-2j89n -- mysql -u wordpress -p -e "show databases"
        Unable to use a TTY - input is not a terminal or the right kind of file
        Enter password: admin
        Database
        information_schema
        performance_schema
        wordpress

Now deploy the wordpress-deployment.

 kubectl apply -f wordpress-deployment.yaml
 
        kubectl apply -f wordpress-deployment.yaml
        service/wordpress created
        persistentvolumeclaim/wp-pv-claim created
        deployment.apps/wordpress created

        kubectl get pods
        NAME                              READY   STATUS    RESTARTS   AGE
        wordpress-78bb764d54-6p5qj        1/1     Running   0          2m12s
        wordpress-mysql-b759dbb45-2j89n   1/1     Running   0          6m20s
