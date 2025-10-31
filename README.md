# TP_hebergement_ssh

**Nom** : Damien Paszkiewicz

## Objectifs

- Comprendre le protocole SSH et son rôle dans l'administration d'un serveur distant.
- Mettre en place un serveur SSH dans un conteneur Docker et s'y connecter depuis un client dédié.
- Manipuler l'authentification par mot de passe puis par clé publique.
- Automatiser la configuration du serveur via un Dockerfile

## Prérequis

- Avoir réalisé le TP d'introduction à Docker (images, conteneurs, ports, volumes).
- Disposer de Docker fonctionnel sur votre machine.
- Savoir utiliser un terminal Bash, zsh , fish ou PowerShell.
- Aucune connaissance préalable de SSH n'est nécessaire

## Résultats attendus

- Deux images Docker ( ssh-server , ssh-client ) construites à partir de vos Dockerfiles.
- Une connexion SSH fonctionnelle entre deux conteneurs reliés par un réseau Docker interne.
- Un rapport court décrivant le protocole SSH, les modes d'authentification et les commandes utilisées.

## Architecture du TP

Vous simulerez une petite infrastructure distante composée de :

- Un conteneur ssh-server basé sur Ubuntu et exposant openssh-server sur le port 22.
- Un conteneur ssh-client utilisé pour initier la connexion SSH.
- Un réseau Docker interne ( ssh-net ) pour relier les deux conteneurs

Un dossier de travail unique est recommandé, par exemple : ~/workspace/tp-hebergement-ssh .

# Étape 0 — Préparer l'environnement Docker

Créez le dossier de travail et placez-vous dedans :

```bash
~/Documents/B3dev/Docker/B3dev-TP_hebergement_ssh$
```

Créez le réseau Docker qui servira de lien entre les conteneurs :

```bash
$ sudo docker network create ssh-net           
8bdd9361b9335b56903012789337c1ff8aac2848f7fd76febc5d3dae9f3ae923
```

Vérifiez la présence du réseau :

```bash
$ sudo docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
7454631caa8c   bridge    bridge    local
69868f0aa484   host      host      local
03a8a6a5f64a   none      null      local
8bdd9361b933   ssh-net   bridge    local
```