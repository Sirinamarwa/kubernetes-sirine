# kubernetes-sirine
# kubernetes-minikube

Minikube est un outil qui permet d'exécuter Kubernetes localement. Minikube fait tourner un cluster Kubernetes à nœud unique sur votre ordinateur personnel (Windows, macOS et Linux) pour tester Kubernetes ou pour le développement quotidien.

## Installation de Docker

### Installation pour Mac, Windows 10 Pro, Enterprise ou Education

https://www.docker.com/get-started

Choisir Docker Desktop

### Installation pour Windows Home

https://docs.docker.com/docker-for-windows/install-windows-home/

## Installation de Kubernetes Minikube

https://minikube.sigs.k8s.io/docs/start/

Minikube fournit un tableau de bord (portail web). Accéder au tableau de bord avec la commande suivante :

```
minikube dashboard
```

## Télécharger ce projet

Ce projet contient un service web codé en Java, mais le langage n'a pas d'importance. Le projet a déjà été compilé et la version binaire est incluse :

Tout d'abord, télécharger et décompresser le projet : https://github.com/charroux/kubernetes-minikube

Vous pouvez aussi utiliser git : `git clone https://github.com/charroux/kubernetes-minikube`

Ensuite, se déplacer dans le sous-répertoire avec `cd kubernetes-minikube/MyService` où se trouve le DockerFile.

## Tester le projet avec Docker

Construire l'image Docker :

```
docker build -t sirine .
```

Vérifier l'image :

```
docker images
```

Démarrer le conteneur :

```
docker run -p 4000:8080 sirine
```

8080 est le port du service web, tandis que 4000 est le port d'accès au conteneur. Tester le service web dans un navigateur : http://localhost:4000 — affiche hello.

Ctrl-C pour arrêter le service web.

Vérifier le containerID :

```
docker ps
```

Arrêter le conteneur :

```
docker stop containerID
```

## Publier l'image sur Docker Hub

Récupérer l'ID de l'image :

```
docker images
```

Tagger l'image Docker :

```
docker tag imageID sirinamarwa/sirine:1
```

Se connecter à Docker Hub :

```
docker login
```

ou

```
docker login http://hub.docker.com
```

ou

```
docker login -u username -p password
```

Pousser l'image sur Docker Hub :

```
docker push sirinamarwa/sirine:1
```

## Créer un déploiement Kubernetes depuis une image Docker

```
kubectl get nodes
```

```
kubectl create deployment myservice --image=sirinamarwa/sirine:1
```

L'image utilisée provient de Docker Hub. Vous pouvez utiliser votre propre image à la place.

Vérifier le pod :

```
kubectl get pods
```

Vérifier que l'état est bien running.

Obtenir les logs complets d'un pod :

```
kubectl describe pods
```

Récupérer l'adresse IP, mais noter que cette adresse est éphémère car un pod peut être supprimé et remplacé par un nouveau.

Ensuite, retrouver le déploiement dans le tableau de bord Minikube. Le conteneur Docker tourne à l'intérieur d'un pod Kubernetes (voir le pod dans le tableau de bord).

Vous pouvez également entrer dans le conteneur en mode interactif avec :

```
kubectl exec -it podname -- /bin/bash
```

où podname est le nom du pod obtenu avec :

```
kubectl get pods
```

Lister le contenu du conteneur avec :

```
ls
```

Ne pas oublier de quitter le conteneur avec :

```
exit
```

## Exposer le déploiement via un Service

Un Service Kubernetes est une abstraction qui définit un ensemble logique de Pods tournant dans le cluster, fournissant tous la même fonctionnalité. À sa création, chaque Service reçoit une adresse IP unique (appelée clusterIP). Cette adresse est liée à la durée de vie du Service et ne changera pas tant que le Service est actif.

## Exposer des routes HTTP et HTTPS depuis l'extérieur du cluster vers les services internes

Pour certaines parties de votre application (par exemple les frontends), vous pouvez exposer un Service sur une adresse IP externe, en dehors de votre cluster.

Les types de Service Kubernetes permettent de spécifier le type de Service souhaité. Le type par défaut est ClusterIP.

Les valeurs de type et leurs comportements :

* ClusterIP : expose le Service sur une IP interne au cluster. Le Service est uniquement accessible depuis l'intérieur du cluster. C'est le ServiceType par défaut.
* NodePort : expose le Service sur l'IP de chaque nœud à un port statique (le NodePort). Un Service ClusterIP, vers lequel le Service NodePort est routé, est automatiquement créé. Le Service NodePort est accessible depuis l'extérieur du cluster via NodeIP:NodePort.
* LoadBalancer : expose le Service en externe via le load balancer d'un fournisseur cloud. Les Services NodePort et ClusterIP, vers lesquels le load balancer externe route, sont automatiquement créés.
* ExternalName : mappe le Service vers le contenu du champ externalName (ex. foo.bar.example.com) en retournant un enregistrement CNAME.

## Exposer une route HTTP et HTTPS avec NodePort

```
kubectl expose deployment myservice --type=NodePort --port=8080
```

Récupérer l'adresse du service :

```
minikube service myservice --url
```

Le format de cette adresse est `NodeIP:NodePort`.

Tester cette adresse dans le navigateur — affiche à nouveau hello.

Retrouver le NodeIP et le NodePort dans le tableau de bord Minikube.

## Mise à l'échelle et load balancing

Vérifier que le déploiement myservice est en cours d'exécution :

```
kubectl get deployments
```

Combien d'instances tournent actuellement :

```
kubectl get pods
```

Démarrer une deuxième instance :

```
kubectl scale --replicas=2 deployment/myservice
```

```
kubectl get deployments
```

et

```
kubectl get pods
```

à nouveau.

## Créer un Service de type LoadBalancer

Vérifier que le déploiement myservice est en cours d'exécution :

```
kubectl get deployments
```

Si un service tourne devant le déploiement, le supprimer d'abord pour en créer un nouveau de type LoadBalancer. Récupérer le service avec :

```
kubectl get services
```

Le supprimer :

```
kubectl delete service serviceName
```

```
kubectl expose deployment myservice --type=LoadBalancer --port=8080
```

```
minikube service myservice --url
```

Tester dans le navigateur.

## Rolling updates

Les rolling updates permettent aux déploiements de se mettre à jour sans interruption de service en remplaçant progressivement les instances de Pods par de nouvelles.

Pour mettre à jour l'image de l'application vers la version 2 :

```
kubectl set image deployments/myservice myservice=sirinamarwa/sirine:2
```

Confirmer la mise à jour :

```
kubectl rollout status deployments/myservice
```

Pour annuler le déploiement et revenir à la dernière version fonctionnelle :

```
kubectl rollout undo deployments/myservice
```

## Créer un déploiement et un service via un fichier YAML

Les fichiers YAML peuvent être utilisés à la place des commandes `kubectl create deployment` et `kubectl expose deployment`.

Appliquer le déploiement :

```
kubectl apply -f myservice-deployment.yml
```

Appliquer le service NodePort :

```
kubectl apply -f myservice-service.yml
```

ou

Appliquer le service de type LoadBalancer :

```
kubectl apply -f myservice-loadbalancing-service.yml
```

Tester ensuite que tout fonctionne comme attendu.

## Routage via Ingress

Ingress n'est pas un type de Service, mais il sert de point d'entrée pour le cluster. Il permet de consolider les règles de routage en une seule ressource, en exposant plusieurs services sous la même adresse IP. Ingress expose des routes HTTP et HTTPS depuis l'extérieur du cluster vers les services internes.

### Configurer Ingress sur Minikube avec le contrôleur NGINX

Activer le contrôleur NGINX Ingress :

```
minikube addons enable ingress
```

Vérifier que le contrôleur NGINX Ingress est en cours d'exécution :

```
kubectl get pods -n ingress-nginx
```

Créer un déploiement et l'exposer en NodePort (pas en LoadBalancer).

Vérifier que ça fonctionne.

Appliquer le fichier ingress :

```
kubectl apply -f ingress.yml
```

Récupérer l'adresse IP d'Ingress :

```
kubectl get ingress
```

Sur Mac : éditer le fichier `/etc/hosts` et ajouter en bas :

`127.0.0.1 myservice.info`

Activer le tunnel Minikube :

```
minikube addons enable ingress-dns
```

```
minikube tunnel
```

Tester dans le navigateur : http://myservice.info/

## Supprimer les ressources

```
kubectl delete services myservice
```

```
kubectl delete deployment myservice
```
