## Objectives

- Namespaces mysql & wordpress
- Secret
- configurations de ressources MySQL et WordPress
    - MySQL 
	    - Service
	    -  PersistentVolumeClaims
	    - Déploiement
    - WordPress
	    - Service
	    - Ingress
	    - PersistentVolumeClaims
	    - Déploiement
- Apply the kustomization directory by `kubectl apply -k ./`

----
<i>Terminologie </i>

<i>Nœud : Une machine de travail dans Kubernetes, faisant partie d'un cluster.</i>

<i>Cluster : Un ensemble de nœuds qui exécutent des applications conteneurisées gérées par Kubernetes. Pour cet exemple, et dans les déploiements Kubernetes les plus courants, les nœuds du cluster ne font pas partie de l'internet public.</i>

<i>Réseau de cluster : Un ensemble de liens, logiques ou physiques, qui facilitent la communication au sein d'un cluster selon le modèle de mise en réseau Kubernetes.</i>

<i>Service : Un service Kubernetes qui identifie un ensemble de Pods à l'aide de sélecteurs d'étiquettes. Sauf mention contraire, les services sont supposés avoir des IP virtuelles uniquement routables au sein du réseau de cluster</i>

----
<h2>Création Namespace,</h2>
Ajouter namespace "wordpress" :
`sudo kubectl create namespace wordpress`

Ajouter namespace "MySQL" :
`sudo kubectl create namespace MySQL`

Afficher les namespaces :
`sudo kubectl get namespaces`

<b>Pour tous les déploiement nous utiliserons le namespace "wordpress"</b>
<h2>Création Secret</h2>
<b>Nous allons créer un dossier nommé 'Secret', où nous stockerons nos secret.</b>

mysqlDB-secret.yaml  :

	apiVersion: v1
	kind: Secret
	metadata:
	  name: mysql-pass
	  annotations:
	    reflector.v1.k8s.emberstack.com/reflection-allowed: "true"
	    reflector.v1.k8s.emberstack.com/reflection-auto-enabled: "true"
	    reflector.v1.k8s.emberstack.com/reflection-allowed-namespaces:"wordpress,mysql"
	type: Opaque
	data:
	  password: cGFzczEyMw==

 ce secret Kubernetes nommé "mysql-pass" contient un mot de passe encodé en base64 ('pass123') et est annoté pour permettre sa réflexion dans d'autres espaces de noms, avec une réflexion automatique activée.

Reflector est un addon Kubernetes conçu pour surveiller les modifications apportées aux ressources (secrets et configmaps) et refléter les modifications apportées aux ressources miroirs dans le même namespaces ou dans d'autres namespaces.

Ajouter reflector :

1. Importer et appliquer le manifest : `sudo kubectl -n kube-system apply -f https://github.com/emberstack/kubernetes-reflector/releases/latest/download/reflector.yaml`
2. Ajouter reflector.v1.k8s.emberstack.com/reflection-allowed : "true" aux annotations de ressources pour permettre la réflexion sur les miroirs.
	Ajouter reflector.v1.k8s.emberstack.com/reflection-allowed-namespaces : "list" aux annotations de la ressource pour autoriser la réflexion uniquement à partir de la liste des namespaces séparés par des virgules.
	Ajoutez reflector.v1.k8s.emberstack.com/reflection-auto-enabled : "true" aux annotations de ressources pour créer automatiquement des miroirs dans d'autres namespaces.

Appliquer le Secret : `sudo kubectl apply -f mysqlDB-secret.yaml`
Voir le secret : `sudo kubectl get secret -n wordpress`

<h2>Ajouter des configurations de ressources pour MySQL et WordPress</h2>
<h3>MySQL ressource configs</h3>
<i>Le manifeste suivant décrit un déploiement MySQL mono-instance.</i>

<b>Service </b>

	apiVersion: v1
	kind: Service
	metadata:
	  name: wordpress-mysql
	  namespace: wordpress
	  labels:
	    app: wordpress
	spec:
	  ports:
	    - port: 3306
	      targetPort: 3306
	  selector:
	    app: wordpress
	    tier: mysql

Le service du nom 'wordpress-mysql' est mis en place afin que wordpress puisse utilisé la Base de donnée en backend.
Le namespace spécifier est wordpress.
Le port mise en place par default pour MySQL est le 3306, je ne l'ai pas changé mais j'ai spécifié un targetPort sur le même port , pour que le service puisse envoyer des requêtes et que le module soit à l'écoute sur ce port.

 <b>PersistentVolumeClaims </b>
 
	apiVersion: v1
	kind: PersistentVolumeClaim
	metadata:
	  name: mysql-pv-claim
	  namespace: wordpress
	  labels:
	    app: wordpress
	spec:
	  accessModes:
	    - ReadWriteOnce
	  resources:
	    requests:
	      storage: 20Gi

MySQL et Wordpress ont tous deux besoin d'un PersistentVolume pour stocker les données. 
Ici j'ai configuré le stockage à hauteur de 20Gi avec un accès en écriture et lecture.

Dans notre environnements de clusters, une classe de stockage (StorageClass) par défaut est installée. (local-path)

`sudo kubectl get StorageClass`

<b>Déploiement </b>

Le conteneur MySQL monte le PersistentVolume dans /var/lib/mysql.
La variable d'environnement MYSQL_ROOT_PASSWORD définit le mot de passe de la base de données à partir du Secret.

	apiVersion: apps/v1
	kind: Deployment
	metadata:
	  name: wordpress-mysql
	  namespace: wordpress
	  labels:
	    app: wordpress
	spec:
	  selector:
	    matchLabels:
	      app: wordpress
	      tier: mysql
	  strategy:
	    type: Recreate
	  template:
	    metadata:
	      labels:
	        app: wordpress
	        tier: mysql
	    spec:
	      containers:
	      - image: public.ecr.aws/docker/library/mysql:8.0
	        name: mysql
	        env:
	        - name: MYSQL_ROOT_PASSWORD
	          valueFrom:
	            secretKeyRef:
	              name: mysql-pass
	              key: password
	        - name: MYSQL_DATABASE
	          value: wordpress
	        - name: MYSQL_USER
	          value: wordpress
	        - name: MYSQL_PASSWORD
	          valueFrom:
	            secretKeyRef:
	              name: mysql-pass
	              key: password
	        ports:
	        - containerPort: 3306
	          name: mysql
	        volumeMounts:
	        - name: mysql-persistent-storage
	          mountPath: /var/lib/mysql
	      volumes:
	      - name: mysql-persistent-storage
	        persistentVolumeClaim:
	          claimName: mysql-pv-claim

Utilisation de la galerie public d'AWS pour pull l'image de mySQL version 8.0 
Le conteneur MySQL monte le PersistentVolume dans /var/lib/mysql. La variable d'environnement MYSQL_ROOT_PASSWORD définit le mot de passe de la base de données à partir du Secret.
MYSQL_DATABASE définit la création de la base de donnée.

Vous pouvez avoir le fichier complet -->[[mysql-deployement]]<--

### Wordpress ressource configs

<i>Le manifeste suivant décrit le déploiement d'une seule instance de WordPress.</i>

<b>Service </b>

	apiVersion: v1
	kind: Service
	metadata:
	  name: wordpress
	  namespace: wordpress
	  labels:
	    app: wordpress
	spec:
	  ports:
	    - port: 80
	  selector:
	    app: wordpress
	    tier: frontend

Service Wordpress exposer sur le port 80.
'tier' indiquant ce service en frontend 
Le service seras appelé par la suite via l'ingress.

<b>Ingress</b>

	apiVersion: networking.k8s.io/v1
	kind: Ingress
	metadata:
	  name: wordpress-ingress
	  namespace: wordpress
	  labels:
	    app: wordpress
	    type: frontend
	  annotations:
	    spec.ingressClassName: "traefik"
	spec:
	  rules:
	  - http:
	      paths:
	      - path: /
	        pathType: Prefix
	        backend:
	          service:
	            name: wordpress
	            port:
	              number: 80

Ingress expose les routes HTTP de l'extérieur du cluster aux services à l'intérieur du cluster. 
spec.ingressClassName définit le Controller à utilisé ici Traefik.
Le path est set up pour que peu importe le chemin définit il va utilisé le service 'wordpress' sur le port 80. 

 <b>PersistentVolumeClaims </b>

	apiVersion: v1
	kind: PersistentVolumeClaim
	metadata:
	  name: wp-pv-claim
	  namespace: wordpress
	  labels:
	    app: wordpress
	spec:
	  accessModes:
	    - ReadWriteOnce
	  resources:
	    requests:
	      storage: 20Gi

Même configuration que le PVC définit pour le déploiement de MySQL.

<b>Déploiement </b>

	apiVersion: apps/v1
	kind: Deployment
	metadata:
	  name: wordpress
	  namespace: wordpress
	  labels:
	    app: wordpress
	spec:
	  selector:
	    matchLabels:
	      app: wordpress
	      tier: frontend
	  strategy:
	    type: Recreate
	  template:
	    metadata:
	      labels:
	        app: wordpress
	        tier: frontend
	    spec:
	      containers:
	      - image: public.ecr.aws/docker/library/wordpress:6.2.1-apache
	        name: wordpress
	        env:
	        - name: WORDPRESS_DB_HOST
	          value: wordpress-mysql
	        - name: WORDPRESS_DB_PASSWORD
	          valueFrom:
	            secretKeyRef:
	              name: mysql-pass
	              key: password
	        - name: WORDPRESS_DB_USER
	          value: wordpress
	        ports:
	        - containerPort: 80
	          name: wordpress
	        volumeMounts:
	        - name: wordpress-persistent-storage
	          mountPath: /var/www/html
	      volumes:
	      - name: wordpress-persistent-storage
	        persistentVolumeClaim:
	          claimName: wp-pv-claim

Utilisation de la galerie public d'AWS pour pull l'image de WordPress version 6.2.1
Le conteneur WordPress monte le PersistentVolume à /var/www/html pour les fichiers de données du site web. 
La variable d'environnement WORDPRESS_DB_HOST définit le nom du service MySQL défini ci-dessus, et WordPress accède à la base de données par le service. 
La variable d'environnement WORDPRESS_DB_PASSWORD définit le mot de passe de la base de données à partir du Secret généré.

Vous pouvez avoir le fichier complet -->[[wordpress-deployement]]<--

<h2>Creation kustomization</h2>

Création fichier kustomization.yaml : 

```shell
cat <<EOF >>./kustomization.yaml
resources:
  - mysql-deployment.yaml
  - wordpress-deployment.yaml
  - Secret/mysqlDB-secret.yaml
EOF
```

Appliquer et Vérifier

`sudo kubectl apply -k ./`

Vous pouvez maintenant vérifier que tous les objets existent.

1. Vérifiez que le Secret existe en exécutant la commande suivante :
    
    ```shell
	sudo kubectl get secrets -n wordpress
    ```

2. Vérifier qu'un PersistentVolume a été provisionné dynamiquement.

```shell
sudo kubectl get pvc
```

3. Vérifiez que le pod fonctionne en exécutant la commande suivante :

```shell
sudo kubectl get pods -n wordpress
```

4. Vérifiez que le service fonctionne en exécutant la commande suivante :

```shell
sudo kubectl get services wordpress -n wordpress
```

5. Exécutez la commande suivante pour obtenir l'adresse IP du service WordPress :

```shell
sudo kubectl port-forward --address 0.0.0.0 wordpress-66d84bc9c4-hgmdj 3030:80 -n wordpress
```

6. Accéder à `172.16.238.14:3030`