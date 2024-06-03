# Récapitulatifs des commandes

Pour WINDOWS nous privilégierons le gestionnaire paquets SCOOP pour certaines installations
(<https://scoop.sh/#/>)

Pour MAC nous passerons par HOMEBREW FORMULAE (<https://formulae.brew.sh/>)

----

## Installation de l'environnement

1 - Installer Kubernetes via docker depuis les paramètres de docker desktop

2 - Installer `kubeclt` la CLI de Kubernetes

- WINDOWS :
  - <https://scoop.sh/#/apps?q=kubectl&id=17eabfd9d69e0947625d6ae59e55e43af3d51663>

```bash
  scoop bucket add main
  scoop install main/kubectl
```

- MAC : (NB : ici `kubectl` s'appelle `kubernetes-cli`)
  - <https://formulae.brew.sh/formula/kubernetes-cli#default>

```bash
  brew install kubernetes-cli
```

3 - Installer `OpenLens` votre interface graphique pour gérer les cluster Kubernetes

- WINDOWS :
  - <https://scoop.sh/#/apps?q=openlens&id=7a01ac6633e4bda71fd7944b5182df4f890eab9c>

```bash
  scoop bucket add extras
  scoop install extras/openlens
```

- MAC :
  - <https://formulae.brew.sh/cask/openlens#default>

```bash
  brew install --cask openlens
```

----

## Instancier un pod via kubectl

```bash
kubectl apply –f <emplacement_fichier>\<nom_de_fichier>
```

----

## Installation d'ArgoCD

via Kubectl depuis votre terminal

- WINDOWS :

```bash
kubectl create namespace argocd 
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

- MAC :

  - <https://formulae.brew.sh/formula/argocd#default>
  
```bash
brew install argocd
```

----

## Connection à argoCD

1 - Activer le port forwarding:

Depuis openLens (catalogue puis cliquer sur votre cluster pour l'instancier):
Workloads --> Pods --> filtre sur le `Namespace:argocd` --> cliquer sur `argocd-server-xxxxx` --> scroll jusque "ports" et activer le `port fowarding` sur le 8080/TCP

Ou en version ligne de commande

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:8080
```

2 - Se connecter:

une fois le port forwarding activer cliquer sur le 8080/TCP pour accéder à l'interface utilisateur.

/!\ votre connection sera en `HTTP` votre navigateur vous bloquera, choississez l'option pour passer outre et se accèder tout de même à la page malgré l'avertissement.

Le user par défaut est : `admin`

----

### Mot de passe Admin principal

- Depuis OpenLens :

Depuis OpenLens (catalogue puis cliquer sur votre cluster pour l'instancier):
Config --> Secrets --> filtre sur `All namespaces` ou `Namespace:argocd` --> cliquer sur `argocd-initial-admin-secret` --> récupérer le mot de passe

- En ligne de commande :

  - WINDOWS :
  
    - version Bash

    ```bash
    kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
    ```

    - version Powershell

    ```Powershell
    kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | %{[System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String($_))}
    ```

  - MAC :

  ```bash
  kubectl -n argocd get secret argocd-initial-admin-secret -o=jsonpath="{.data.password}" | base64 --decode
  ```

----

## Créer une connection entre ArgoCD et votre repo Github

1 - Dans votre compte Github :

- Aller dans les `settings` de votre compte
- Puis `Developper Settings` (<https://github.com/settings/apps>) --> `Personnal access tokens` --> `fine-grained token` --> Generate new token

- Choisissez un nom, une expiration
- Dans la partie `Repository access`, la 3eme options et cibler le nom de votre repository de travail

- Puis dans `Permissions` --> `Repository permissions`rRemplissez l’ensemble des permissions, pour les besoins du support mettre toutes les autorisations au max (read and write ou read-only)

  !\ votre token une fois généré il ne sera visible qu'à ce moment `sauvegardez le si nécessaire`

2 - Depuis ArgoCD

Création d'une connexion au repo (`settings` --> `repositories`) :

- Sélectionnez la méthode https (puis ajoutez l'URL de votre repo)
- Mettez votre nom de compte Github
- A la place du mot de passe saisissez le token

--> argoCD peut maintenant lire les fichiers de déploiements (manifiests) de votre repo Github

----

## Créer une pipeline CD avec ArgoCD

Dépendant de l'emplacement de votre fichier remplissez le `PATH`

-> remplissez avec juste `.` si votre fichier de déploiement est situé à la racine
-> sinon indiquez le chemin ./emplacement_1/emplacement_2/...

/!\ Si votre manifest est dans un répertoire pensez à cocher la fonction `Directory Recurse` afin d'indiquer à ArgoCd qu'il doit lire un fichier dans un répertoire.

General :
  Application Name : un nom en minuscule

Project Name : default

Sync Policy : Manual ou Automatic

Destination:

- <https://kubernetes.default.svc> --> noeud par défaut
- Namespace : à définir

----

## Création d'un utilisateur en modifiant le ConfigMap

- Vous pouvez modifier le ConfigMap argocd-cm dans le namespace ArgoCD pour ajouter un utilisateur

1 - Depuis openLens CONFIG --> CONFIGMAP --> argocd-cm --> cliquer sur le bouton d'édition

2 - Ajoutez ces lignes puis saugarder

```bash
data:
 accounts.<nom_utilisateur>: login, apiKey
 accounts.<nom_utilisateur>.enabled: "true"
```

----

## Installation d'ArgoCD-CLI

- WINDOWS :

```bash
scoop install main/argocd
```

- MAC :

Déjà fait `:)`

----

## Connection à ArgoCD depuis d'ArgoCD-CLI

```bash
argocd login localhost:<port_présent_dans_l'URL_de_votre_navigateur>
```

- Ex:

```bash
argocd login localhost:8080 # Connexion au serveur argocd avec votre identifiant et mot de passe admin
```

----

## Setup d'un mdp de compte utilisateur depuis d'ArgoCD-CLI

```bash
argocd account update-password --account <nom_du_rôle>
```

----

## Ajout de rôle à compte utilisateur depuis d'ArgoCD-CLI

```bash
argocd proj role add-group <nom-du_groupe> <nom_du_compte> <nom_du_rôle>
```

- Ex:

```bash
argocd proj role add-group pipelinecicd developpeur developpeur
```

----

## Commandes Utiles ArgoCD

```bash
# Liste des applications
argocd app list

# Détail d'une application
argocd app get <nom_application>

# Liste des projets
argocd proj list

# Détail d'un projet
argocd proj get <nom_projet>
```

----

## Instalation d'argocd-autopilot

/!\ Pensez à reset votre cluster Kubernetes depuis les paramètres de docker desktop et le réinstaller
/!\ A la fin de l'installation pensez à sauvegarder votre mot de passe admin qui apparait dans votre terminal (avant dernière ligne)

- WINDOWS :

  ```bash
  Scoop install main/argocd-autopilot
  ```

  ```bash
  $env:GIT_REPO=“<votre_adresse_de_repo_pour_l'installation>”
  ```

  ```bash
  $env:GIT_TOKEN=“<votre_fine_grained_token_de_ce_repo>”
  ```

  ```bash
  argocd-autopilot repo bootstrap
  ```

  ```bash
  kubectl port-forward -n argocd svc/argocd-server 8080:80
  ```

- MAC :

  ```bash
  brew install argocd-autopilot
  ```

  ```bash
  export GIT_REPO=“<votre_adresse_de_repo_pour_l'installation>”
  ```

  ```bash
  export GIT_TOKEN=“<votre_fine_grained_token_de_ce_repo>”
  ```

  ```bash
  argocd-autopilot repo bootstrap
  ```

  ```bash
  kubectl port-forward -n argocd svc/argocd-server 8080:80
  ```

----

## Connexion à d'argocd-autopilot

Depuis OpenLens refaite un port forwarding comme pour ArgoCD classique et cliquer sur le lien (passer outre l'avertisement de connexion en HTTP de bvotre navigateur)

----

## Création d'un projet et d'un déploiement test avec argocd-autopilot

création du projet

  ```bash
-argocd-autopilot project create testing
```

création du déploiement dans le projet

```bash
argocd-autopilot app create hello-world --app github.com/argoproj-labs/argocd-autopilot/examples/demo-app/ -p testing --wait-timeout 2m
```
