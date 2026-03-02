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
