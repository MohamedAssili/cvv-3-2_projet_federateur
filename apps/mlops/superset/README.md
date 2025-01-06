Cette partie est pour gérer les sources pour le sous-projet superset.

Documentation sur Apache Superset

Introduction
Apache Superset est une plateforme de visualisation et d'exploration de données open-source. Conçue pour être intuitive, puissante et évolutive, elle permet aux utilisateurs de créer et partager des tableaux de bord interactifs, explorer des ensembles de données et exécuter des requêtes SQL. Apache Superset est compatible avec une large gamme de bases de données SQL et d'entrepôts de données.
Caractéristiques principales
•	Support multi-base de données : Compatibilité avec les bases de données courantes telles que PostgreSQL, MySQL, MariaDB, BigQuery, Snowflake, et bien d'autres.
•	Exploration sans code : Interface utilisateur intuitive permettant de créer des visualisations sans avoir besoin d'écrire du code.
•	Requêtes SQL avancées : Interface pour l'écriture et l'exécution de requêtes SQL, avec des outils d'analyse des résultats.
•	Extensibilité : Une architecture modulaire permettant des intégrations personnalisées et l'ajout de nouveaux types de visualisations.
•	Sécurité et gestion des accès : Contrôle granulé des permissions pour les utilisateurs et les rôles.

I-	Installation et configuration d'Apache Superset dans Kubernetes

Pour installer Apache Superset sur Kubernetes, vous pouvez suivre ces étapes. Apache Superset est une application Web de business intelligence moderne et adaptée aux entreprises que vous pouvez déployer à l'aide de graphiques Helm ou en créant manuellement des ressources Kubernetes. Vous trouverez ci-dessous un guide pour la méthode manuel.
Installer Superset dans Kubernetes implique plusieurs étapes, notamment la préparation du cluster, la configuration de la base de données backend et l'intégration des composants nécessaires.


________________________________________
Etape 1: Déployer Apache Superset manuellement :

Préparer les ressources Kubernetes Créez des ressources Kubernetes telles que Deployment, Service et PersistentVolumeClaim (PVC).
Le super-ensemble de configuration de base de données nécessite une base de données de métadonnées (par exemple, PostgreSQL ou MySQL). 
Déploiement d’une base de données PostgreSQL dans :

apiVersion: apps/v1 
kind: Deployment 
metadata: 
  name: superset-postgres 
spec: 
  replicas: 1 
  selector: 
    matchLabels: 
      app: superset-postgres 
  template: 
    metadata: 
      labels: 
        app: superset-postgres 
    spec: 
      containers: 
      - name: postgres 
        image: postgres:13 
        env: 
        - name: POSTGRES_DB 
          value: superset 
        - name: POSTGRES_USER 
          value: superset_user 
        - name: POSTGRES_PASSWORD 
          value: superset_password 
        ports: 
        - containerPort: 5432

Déploiement de Superset : Créer un manifeste de déploiement de Superset:
apiVersion: apps/v1 
kind: Deployment 
metadata: 
  name: superset 
spec: 
  replicas: 1 
  selector: 
    matchLabels: 
      app: superset 
  template: 
    metadata: 
      labels: 
        app: superset 
    spec: 
      containers: 
      - name: superset 
        image: apache/superset:latest 
        env: 
        - name: SUPERSET_SECRET_KEY 
          value: "your_secret_key" 
        - name: SUPERSET_DATABASE_URI 
          value: "postgresql+psycopg2://superset_user:superset_password@superset-postgres/superset" 
        ports: 
        - containerPort: 8088 
        command: 
        - "/bin/sh" 
        - "-c" 
        - | 
          superset db upgrade && 
          superset init && 
          gunicorn -w 2 -b 0.0.0.0:8088 "superset.app:create_app()"

Créer un Service : Exposer le déploiement de Superset à l'aide d'un Service:
apiVersion: v1 
kind: Service 
metadata: 
  name: superset 
spec: 
  selector: 
    app: superset 
  ports: 
  - protocol: TCP 
    port: 80 
    targetPort: 8088 
  type: LoadBalancer


Appliquer les ressources : Déployer les fichiers YAML:
kubectl apply -f superset-postgres.yaml
kubectl apply -f superset.yaml
kubectl apply -f superset-service.yaml

Accéder à Superset : Obtenir l'IP externe du service Superset:
kubectl get svc superset
Ouvrez votre navigateur et accédez à l'IP externe. ________________________________________

Par défaut, lorsque vous installez Apache Superset, il n'y a pas d'utilisateur préconfiguré. Vous devez créer un utilisateur administrateur avant de pouvoir vous connecter. Voici les étapes pour créer un utilisateur administrateur et configurer le mot de passe :
________________________________________
Étape 2 : Créer un utilisateur administrateur
1.	Exécutez une commande pour accéder à votre pod Superset :
kubectl exec -it <nom-du-pod-superset> -- bash
Remplacez <nom-du-pod-superset> par le nom de votre pod Superset. Vous pouvez lister les pods avec :
kubectl get pods
2.	Une fois dans le conteneur, créez un utilisateur administrateur avec la commande suivante :
superset fab create-admin
Cette commande vous demandera les informations suivantes :
	Username : Entrez un nom d'utilisateur (ex. admin).
	User first name : Prénom de l'utilisateur.
	User last name : Nom de famille de l'utilisateur.
	Email : Adresse e-mail.
	Password : Définissez un mot de passe.
________________________________________

Etape 3: Lancement de l’application:

Si vous exécutez Kubernetes localement (par exemple, avec Minikube, Kind ou MicroK8s), l'utilisation du type de service LoadBalancer ne fonctionnera pas, sauf si vous avez un fournisseur de load balancer configuré. La plupart des configurations Kubernetes locales n'ont pas de load balancer natif, donc vous devrez trouver une autre méthode pour exposer le service.
Voici quelques options pour exécuter votre service localement :
________________________________________
1. Utiliser l'émulation du LoadBalancer de Minikube :
Si vous utilisez Minikube, vous pouvez exposer le service LoadBalancer en exécutant la commande suivante :
minikube service superset
Cela ouvrira l'interface utilisateur de Superset dans votre navigateur. Minikube émule le comportement du LoadBalancer pour le développement local.
________________________________________
2. Utiliser Port Forwarding
Vous pouvez rediriger le port du service vers votre machine locale :
kubectl port-forward svc/superset 8088:80
Vous pouvez maintenant accéder à Superset à l'adresse http://localhost:8088.

________________________________________
Étape 4 : Vérifier et initialiser la base de données (si nécessaire)
Si ce n'est pas déjà fait, assurez-vous d'avoir initialisé la base de données de Superset :
superset db upgrade
Puis, appliquez les rôles par défaut et configurez l'application :
superset init
________________________________________
Étape 5 : Connexion à Superset
1.	Une fois l'utilisateur créé, accédez à l'interface web de Superset à l'adresse correspondante, par exemple :
http://192.168.49.2:30775/superset/welcome/
ou
http://<adresse-ip-du-service-superset>:<port>
2.	Connectez-vous avec le username et le password que vous avez définis.
________________________________________
Étape 6 (Optionnel) : Réinitialiser le mot de passe
Si vous oubliez votre mot de passe, vous pouvez le réinitialiser en exécutant la commande suivante dans le pod Superset :
superset fab reset-password --username <votre-nom-utilisateur>
Ensuite, entrez un nouveau mot de passe.
________________________________________




- https://superset.apache.org/
- https://github.com/apache/superset/tree/master
