# Installation du client fluxctl
        
        brew install fluxctl

# Installation du flux-system

1. Cloner ce projet
2. Déplacer vous au niveau du dossier ```cloud-gitops```
2. Installer le flux-system avec la commande suivante

        kubectl apply -k install/

3. Récuperer la clé de deploiement ssh avec la commande suivante

        fluxctl --k8s-fwd-ns=flux-system identity

4. Avec cette clé, creer un clé de deploiement ssh au niveau de ce repos avec l'option read/write

# Integration d'un microservice

## I. Au niveau du repos du flux-system (ce repos)

1. cloner ce repos
2. taper la commande suivante pour rendre le script executable

        chmod +x scripts/create-team.sh

3. Creer le projet avec la commande suivante

        ./scripts/create-team.sh <nom du projet>

NB. Cette commande créera tous les fichiers nécessaires pour nous.

4. Au niveau du fichier ```cluster/<nom du projet>/flux-patch.yaml```, changer l'url git pour que ça point vers le repo du nouveau projet.
    
```bash
vim cluster/<nom du projet>/flux-patch.yaml

--git-url=git@github.com:org/dev-cluster
```

5. Faire un commit et pusher le faire le vers le referentiel

6. Attendre quelques minutes pour que flux crée tous les objets kubernetes se trouvant dans le dossier ```cluster/<nom du projet>``` ou faites le vous meme avec la commande suivante:
    
        kubectl apply -k cluster/<nom du projet>

7. Recuperer la clé ssh de la nouvelle instance flux associée au projet avec la commande suivante

        fluxctl --k8s-fwd-ns=<nom du projet> identity

8. Avec cette clé, creer un clé de deploiement ssh au niveau du repos du nouveau projet avec l'option read/write

## II. Au niveau du repo du projet
creer: 
1. un fichier nommé ```flux.yaml``` avec le contenu suivant.
```yaml
version: 1
patchUpdated:
  generators:
    - command: kustomize build .
  patchFile : flux-patch.yaml
```
2. un fichier vide nommé ```flux-patch.yaml```
3. un autre fichier nommé ```kustomization.yaml``` avec le contenu suivant
```yaml
namespace: team1
bases:
  - ./workloads/
```
4. Et en fin creer le dossier ```workloads``` ou vous allez mettre vos manifestes kubernetes.

#### NB. 
Flux fera un un ```kubectl apply -k workloads``` à chaque commit sur la branch master

# Commandes flux utiles
Exportation de la variable d'environnement contenant le namespace de l'instance flux à utiliser:

    export FLUX_FORWARD_NAMESPACE=team1

Si on veut utiliser une instance de flux qui se trouve sur un namespace autre que celui par defaut, il faut passer l'option --k8s-fwd-ns au niveau de la commande. Par exemple:

    fluxctl list-workloads --k8s-fwd-ns=team2

## Synchronisation
Flux verifie périodiquement s'il y'a de nouveaux changements sur le référentiel. Ce qui fait qu'il peut y'avoir une latence de quelques secondes voire minutes pour que flux synchronise l'etat du cluster avec le repo.
On peut forcer la synchronisation avec la commande suivante

                fluxctl sync

## Lister tous les workloads:

    fluxctl list-workloads --all-namespaces
  
Filtrage par neamespace
    
    fluxctl list-workloads -n team1

## Lister toutes les images

    fluxctl list-images

Filtrage par workload
    
    fluxctl list-images --workload team1:deployment/backend
  
## Changer la version de l'image d'un workload (faudrait que le déploiement automatique des images soit désactiver)

    fluxctl release --workload=<workload> --update-image=<image>:<tag>

## Activer le déploiement automatique des nouvelles images d'un workload

    fluxctl automate --workload=<nom du workload>

## Désactiver le déploiement automatique des nouvelles images d'un workload

    fluxctl deautomate --workload=<nom du workload>
## Verrouiller un workload

    fluxctl lock --workload=deployment/helloworld
    
## Enlever le verrou d'un workload

    fluxctl lock --workload=deployment/helloworld
