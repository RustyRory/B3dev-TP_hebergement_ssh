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

# Étape 1 — Construire l'image du serveur SSH

Créez un fichier `Dockerfile.server` contenant :

```bash
FROM ubuntu:24.04
RUN apt update \
&& apt install -y openssh-server sudo \
&& mkdir /var/run/sshd
# Crée un utilisateur non-root "student"
RUN useradd -m student \
&& echo "student:password" | chpasswd \
&& adduser student sudo
# Active l'authentification par mot de passe (temporaire pour le TP)
RUN sed -i 's/#PasswordAuthentication yes/PasswordAuthentication yes/' /etc/ssh/sshd_config \
&& sed -i 's/PermitRootLogin prohibit-password/PermitRootLogin no/' /etc/ssh/sshd_config
EXPOSE 22
CMD ["/usr/sbin/sshd", "-D"]
```

Construisez l'image Docker du serveur :

```bash
$ sudo docker build -t ssh-server -f Dockerfile.server .
[+] Building 33.7s (8/8) FINISHED                                                                                                                 docker:default
 => [internal] load build definition from Dockerfile.server                                                                                                 0.0s
 => => transferring dockerfile: 565B                                                                                                                        0.0s
 => [internal] load metadata for docker.io/library/ubuntu:24.04                                                                                             1.4s
 => [internal] load .dockerignore                                                                                                                           0.0s
 => => transferring context: 2B                                                                                                                             0.0s
 => [1/4] FROM docker.io/library/ubuntu:24.04@sha256:66460d557b25769b102175144d538d88219c077c678a49af4afca6fbfc1b5252                                       1.8s
 => => resolve docker.io/library/ubuntu:24.04@sha256:66460d557b25769b102175144d538d88219c077c678a49af4afca6fbfc1b5252                                       0.0s
 => => sha256:66460d557b25769b102175144d538d88219c077c678a49af4afca6fbfc1b5252 6.69kB / 6.69kB                                                              0.0s
 => => sha256:d22e4fb389065efa4a61bb36416768698ef6d955fe8a7e0cdb3cd6de80fa7eec 424B / 424B                                                                  0.0s
 => => sha256:97bed23a34971024aa8d254abbe67b7168772340d1f494034773bc464e8dd5b6 2.30kB / 2.30kB                                                              0.0s
 => => sha256:4b3ffd8ccb5201a0fc03585952effb4ed2d1ea5e704d2e7330212fb8b16c86a3 29.72MB / 29.72MB                                                            0.9s
 => => extracting sha256:4b3ffd8ccb5201a0fc03585952effb4ed2d1ea5e704d2e7330212fb8b16c86a3                                                                   0.7s
 => [2/4] RUN apt update && apt install -y openssh-server sudo && mkdir /var/run/sshd                                                                      28.5s
 => [3/4] RUN useradd -m student && echo "student:password" | chpasswd && adduser student sudo                                                              0.4s 
 => [4/4] RUN sed -i 's/#PasswordAuthentication yes/PasswordAuthentication yes/' /etc/ssh/sshd_config && sed -i 's/PermitRootLogin prohibit-password/Permi  0.2s 
 => exporting to image                                                                                                                                      1.3s 
 => => exporting layers                                                                                                                                     1.3s 
 => => writing image sha256:2526a34a3d53f692492d246991a7ee6d4a0b1dcdee841cd99a74ccb94f0b8fcc                                                                0.0s
 => => naming to docker.io/library/ssh-server                                    
```

 Lancez le conteneur serveur en arrière-plan :

```bash
$ sudo docker run -d --name ssh-server --network ssh-net ssh-server
c79f65000183271437044f7b0ba8b8fd85454cd911035ca3d0edeb21450d24db
```

Vérifiez que le service SSH est bien démarré :

```bash
$ sudo docker exec -it ssh-server ps aux | grep sshd
root           1  0.0  0.0  12020  7936 ?        Ss   15:12   0:00 sshd: /usr/sb
```

# Étape 2 — Construire l'image du client SSH

1. Créez un fichier `Dockerfile.client` :

```bash
FROM ubuntu:24.04
RUN apt update \
&& apt install -y openssh-client vim iputils-ping
WORKDIR /root
CMD ["bash"]
```

2. Construisez puis lancez le conteneur client :

```bash
$ sudo docker build -t ssh-client -f Dockerfile.client .
[+] Building 16.0s (7/7) FINISHED                                                                                                                 docker:default
 => [internal] load build definition from Dockerfile.client                                                                                                 0.0s
 => => transferring dockerfile: 155B                                                                                                                        0.0s
 => [internal] load metadata for docker.io/library/ubuntu:24.04                                                                                             0.7s
 => [internal] load .dockerignore                                                                                                                           0.0s
 => => transferring context: 2B                                                                                                                             0.0s
 => CACHED [1/3] FROM docker.io/library/ubuntu:24.04@sha256:66460d557b25769b102175144d538d88219c077c678a49af4afca6fbfc1b5252                                0.0s
 => [2/3] RUN apt update && apt install -y openssh-client vim iputils-ping                                                                                 14.0s
 => [3/3] WORKDIR /root                                                                                                                                     0.0s 
 => exporting to image                                                                                                                                      1.2s 
 => => exporting layers                                                                                                                                     1.1s 
 => => writing image sha256:6889010dc65789c9153bc864ab32ff029808fe0f61158e336a76d0b4fafae5fa                                                                0.0s 
 => => naming to docker.io/library/ssh-client                         
```

```bash
$ sudo docker run -it --rm --network ssh-net --name ssh-client ssh-client            
root@2f7570d214d0:~# 
```

3. Depuis le terminal du client, vérifiez la connectivité réseau :

```bash
root@2f7570d214d0:~# ping -c 2 ssh-server
PING ssh-server (172.18.0.2) 56(84) bytes of data.
64 bytes from ssh-server.ssh-net (172.18.0.2): icmp_seq=1 ttl=64 time=0.222 ms
64 bytes from ssh-server.ssh-net (172.18.0.2): icmp_seq=2 ttl=64 time=0.053 ms

--- ssh-server ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1001ms
rtt min/avg/max/mdev = 0.053/0.137/0.222/0.084 ms
root@2f7570d214d0:~# 
```

# Étape 3 — Connexion SSH par mot de passe

Depuis le terminal du client, initiez une connexion SSH :

```bash
root@2f7570d214d0:~# ssh student@ssh-server
The authenticity of host 'ssh-server (172.18.0.2)' can't be established.
ED25519 key fingerprint is SHA256:EIaFXNDSzeEFV5NPVhpVl5FHGZYj0BfpCFWuqIc6uio.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'ssh-server' (ED25519) to the list of known hosts.
student@ssh-server's password: : 
Welcome to Ubuntu 24.04.3 LTS (GNU/Linux 6.8.0-87-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

This system has been minimized by removing packages and content that are
not required on a system that users do not log into.

To restore this content, you can run the 'unminimize' command.

The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

$ 
```

Vérifiez que vous êtes bien connecté sur le serveur :

```bash
$ whoami
student
$ hostname
c79f65000183
```

