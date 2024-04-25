# Installation d'ArgoCD 

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

# Installation de l'interface web d'ArgoCD

> Cette étape peut être fait par l'interface graphique d'openlens en se rendant la partie Network filtré sur le namespace argocd et en cliquant sur le service argocd-server-xxxx et en cliquant sur le bouton "Port Forward"

> Ou en version ligne de commande
```bash 
kubectl port-forward svc/argocd-server -n argocd 8080:8080
```

# Connexion à l'interface web d'ArgoCD

> version bash
```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

> version powershell
```powershell
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | %{[System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String($_))}
```

# Installation du CLI d'ArgoCD

```powershell
scopp add main # si vous n'avez pas encore installé le bucket main
scoop install main/argocd
``` 

# Commande ArgoCD utile pour le cours 

```bash
argocd login localhost:8080 # Connexion au serveur argocd avec votre identifiant et mot de passe admin
argocd app list # Liste des applications
argocd app get <nom_application> # Détail d'une application
argocd proj list # Liste des projets
argocd proj get <nom_projet> # Détail d'un projet
```

# Création d'un utilisateur en modifiant le configMap

> Vous pouvez modifier le configMap argocd-cm dans le namespace argocd pour ajouter un utilisateur
> Il faudra rajouter les lignes suivantes dans le fichier
```bash
data:
    accounts.<nom_utilisateur>: login, apiKey
    accounts.<nom_utilisateur>.enabled: "true"
```

# Ajoutez un rôle à un utilisateur
```bash
argocd proj role add-group pipelinecicd developpeur developpeur
```

# Changer le mot de passe d'un utilisateur
```bash
argocd account update-password <nom_utilisateur>
```