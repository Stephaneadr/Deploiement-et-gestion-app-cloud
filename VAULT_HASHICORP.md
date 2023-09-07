Install Helm :

```
curl -LO https://get.helm.sh/helm-v3.7.2-linux-amd64.tar.gz
tar -C /tmp/ -zxvf helm-v3.7.2-linux-amd64.tar.gz
rm helm-v3.7.2-linux-amd64.tar.gz
mv /tmp/linux-amd64/helm /usr/local/bin/helm
chmod +x /usr/local/bin/helm
```

Ajoutons les dépôts Helm, afin d'accéder aux manifestes Kubernetes.

```
helm repo add hashicorp https://helm.releases.hashicorp.com
```

installé Vault:

```
helm search repo hashicorp/vault
```

<h4>initialising Vault</h4>

```
sudo kubectl exec -it vault-0 -- sh
	vault operator init
	vault operator unseal
```

Vérifier que le POD run :

```
sudo kubectl get po
```

1. Enable the database secrets engine
	`sudo kubectl exec -it vault-0 -- sh`
	 `vault secrets enable database`
	 
2. Configure Vault with the proper plugin and connection information:
	`vault write database/config/my-mysql-database \
    `plugin_name=mysql-database-plugin \
    `connection_url="{{username}}:{{password}}@tcp(10.42.2.68:3306)/" \
    `allowed_roles="*" \
    `username="root" \
    `password="pass123"
    
3. Rotation Pass
	 ``vault write -force database/rotate-root/my-mysql-database
	 
4. Créer un rôle statique
	`tee rotation.sql <<EOF ALTER USER "{{name}}" WITH PASSWORD '{{password}}'; EOF
	
5. Création rôle statique nommé :
	````
	vault write database/static-roles/sql-role \
    db_name=my-mysql-database \
    rotation_statements=@rotation.sql \
    username="root" \
    rotation_period=604800
``

6. Vérifier la configuration de sql-role

```shell-session
$ vault read database/static-roles/sql-role
```

7. Créez un fichier nommé **`apps.hcl`** avec la politique suivante.

```shell-session
tee apps.hcl <<EOF
path "database/static-creds/education" {
  capabilities = [ "read" ]
}
EOF
```

8. Créez une politique nommée `apps` en utilisant le fichier `apps.hcl`.
	`vault policy write apps apps.hcl
	
9. Récupération Cred 
	`VAULT_TOKEN=$APPS_TOKEN vault read database/static-creds/sql-role