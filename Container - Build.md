# dotNet Core on Kubernetes

Application ASP.NET Core déployé en local dans un cluster Kubernetes à node unique


# 2. Compilation et génération d'image docker

Apres avoir défini le fichier Dockerfile du projet, il s'agit d'envoyer les images docker dans un registre de conteneur:

## A.Prérequis

Les prérequis suivants doivent etre satisfaits:
- docker desktop

## B.Etapes

1. Se connecter au portail web de dockerhub: [https://hub.docker.com/](https://hub.docker.com/)
2. Aller sur "Create repository"
3. Choisir un nom et une description pour le repository et procéder à la création
4. Se logger sur dockerhub depuis la ligne de commande
```bash
$ docker login --username=[nomutilisateur] --email=[votre@email.com]
````
Entrer le mot de passe si demandé

5. Générer l'image docker
```bash
$ docker build -t [nomutilisateur]/[nomrepository]:[nomtag] -t [nomutilisateur]/[nomrepository]:[latest] .
````

6. Vérifier l'image généré

```bash
$ docker images
````

7. Tagger l'image docker [nomutilisateur]/[nomrepository]:[nomtag]
```bash
$ docker tag [imageID] [tagname]
````

8. Pousser l'image vers le repository dockerhub
```bash
$ docker push [nomutilisateur]/[nomrepository]:[latest]
````
