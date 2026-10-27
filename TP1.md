# TP1 : Jouer le rôle d'un fournisseur IaaS en local

## Filière RIST

### Groupe de TP

**Étudiant 1 : CISSE FATIM

**Étudiant 2 :** **DOHOU DEBORAH ELIANE 

**Étudiant 3 :** **KRI BANTI HERMES 

**Date :** 

**Enseignant :** **Dr BAKAYOKO

---

# Introduction

L'objectif de ce TP est de simuler le rôle d'un fournisseur **IaaS (Infrastructure as a Service)** en local à l'aide de l'outil **Multipass**.

Nous allons créer deux machines virtuelles Ubuntu :

- une machine **web** jouant le rôle de serveur ;
- une machine **client** jouant le rôle d'utilisateur.

Le TP consistera à :

- créer les machines virtuelles ;
- vérifier leur connectivité réseau ;
- déployer un serveur web Nginx ;
- contrôler l'accès au serveur grâce au pare-feu UFW ;
- mesurer les ressources utilisées par la machine virtuelle.


# Partie A : Création et communication entre les machines virtuelles

Les machines virtuelles ont été créées et administrées à l'aide de Multipass depuis PowerShell.

## 1. Vérification de Multipass

Nous avons d'abord vérifié l'installation de Multipass.
### Commande utilisée
 
```bash
multipass version
```
### Résultat

```Text
multipass 1.16.4+win
multipassd 1.16.4+win
```
Le logiciel Multipass est correctement installé et opérationnel.

---

## 2. Création de la machine virtuelle web

### Commande

```bash
multipass launch --name web
```
### Résultat

```Text
Launched: web
```
La machine virtuelle **web** a été créée avec succès.

---

## 3. Création de la machine virtuelle client

### Commande

```bash
multipass launch --name client
```

### Résultat

```Text
Launched: client
```

La machine virtuelle **client** a été créée avec succès.

---

## 4. Vérification des machines créées

### Commande

```bash
multipass list
```

### Résultat

| Machine | État    | Adresse IP    |
| ------- | ------- | ------------- |
| web     | Running | 172.25.127.94 |
| client  | Running | 172.25.118.57 |
Les deux machines sont allumées et disposent chacune d'une adresse IP.

---

## 5. Identification des machines

### Machine web

```shell
hostname
```

### Résultat

```Text
web
```

### Adresse IP

```Shell
ip -4 addr
```

### Résultat

```Text
172.25.127.94
```

---

### Machine client

Adresse IP détectée :

```Text
172.25.118.57
```
---

## 6. Test de connectivité réseau

Depuis la machine **client**, nous avons testé la communication avec la machine **web**.

### Commande

```bash
ping -c 4 172.25.127.94
```

### Résultat

```Text
4 packets transmitted
4 received
0% packet loss
```

### Analyse

Le test ping a réussi.

Cela montre que :

- les deux machines sont reliées au réseau virtuel ;
- le routage fonctionne correctement ;
- les paquets ICMP circulent entre les deux machines.

Le réseau créé par Multipass est donc opérationnel.

---

# Partie B : Publication, protection et mesure des ressources

---

# 1. Installation du serveur web Nginx

Connexion à la machine web :

```bash
multipass shell web
```

Mise à jour des paquets :

```bash
sudo apt update
```

Installation de Nginx :

```bash
sudo apt install nginx -y
```

---

## Vérification du service

### Commande

```bash
systemctl status nginx
```

### Résultat

```Text
active (running)
```
### Analyse

Le service Nginx est démarré correctement et est prêt à recevoir des connexions HTTP.

---

# 2. Test du serveur web

Depuis la machine client :

### Commande

```bash
curl http://172.25.127.94
```


### Résultat

Le contenu de la page par défaut Nginx est affiché :

<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>

### Analyse

Ce résultat confirme que :

- le serveur Nginx fonctionne ;
- le client peut accéder au service web ;
- la communication réseau entre les deux machines est correcte.
---
# 3. Sécurisation avec le pare-feu UFW

## Activation du pare-feu

### Commande

```Shell
sudo ufw enable
```

### Résultat

```Text
Firewall is active and enabled on system startup
```
Le pare-feu est désormais actif.

---

## Autorisation de SSH

Afin de conserver l'accès distant au serveur :

```Shell
sudo ufw allow 22/tcp
```

### Résultat

```Text
Rule added
```
---

## Blocage du port HTTP

### Commande

```Shell
sudo ufw deny 80/tcp
```

### Résultat

```Text
Rule added
```
---

## Vérification des règles

### Commande

```Shell
sudo ufw status numbered
```

### Résultat

```Text
22/tcp ALLOW
80/tcp DENY
```
---

# 4. Vérification du blocage

Depuis le client :

### Commande

```Shell
curl http://172.25.127.94
```
### Résultat

```Text
curl: (28) Failed to connect
```
### Analyse

La connexion HTTP échoue car le pare-feu bloque désormais le port 80.
Cette étape démontre l'efficacité de la politique de filtrage mise en place avec UFW.

---

# 5. Réouverture du port HTTP

### Commande

```Shell
sudo ufw allow 80/tcp
```

### Résultat

```Text
Rule updated
```
---

## Vérification

```Shell
sudo ufw status
```

### Résultat

```Text
22/tcp ALLOW
80/tcp ALLOW
```
---

## Nouveau test depuis le client

### Commande

```Shell
curl http://172.25.127.94
```
### Résultat

<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
### Analyse

L'accès au serveur web est de nouveau possible.

Le service est publié correctement sur le réseau.

---

# 6. Mesure des ressources système

Mesures effectuées sur la machine virtuelle **web**.

---

## Mémoire RAM

### Commande

```Shell
free -h
```
### Résultat

```Text

Mémoire totale : 890 MiB
Mémoire utilisée : 364 MiB
Mémoire disponible : 525 MiB
```
### Analyse

La machine virtuelle dispose d'environ 1 Go de mémoire RAM, ce qui est suffisant pour héberger un serveur web léger.

---

## Espace disque

### Commande

```Shell
df -h /
```
### Résultat

```Text
Taille totale : 3,8 Go
Utilisé : 2,2 Go
Disponible : 1,6 Go
Occupation : 58 %
```
### Analyse

Le système dispose encore d'un espace disque libre suffisant pour accueillir des applications supplémentaires.

---

## Nombre de processeurs

### Commande

```Shell
nproc
```
### Résultat

```Text
1
```
### Analyse

La machine virtuelle dispose d'un processeur virtuel.

Cette configuration est adaptée à un petit service web de démonstration.

---

# Architecture mise en œuvre

                    Réseau Multipass

      +-------------------------------+
      |                               |
      |       172.25.x.x              |
      |                               |
      +-------------------------------+

             |                 |
             |                 |
             |                 |
             v                 v

      +---------------+   +---------------+
      |    client     |   |      web      |
      |172.25.118.57  |   |172.25.127.94  |
      +---------------+   +---------------+
                                  |
                                  |
                                  v

                           +------------+
                           |   Nginx    |
                           | Port 80    |
                           +------------+


# Conclusion

Ce TP nous a permis de comprendre concrètement le principe d'un fournisseur **IaaS**.

Grâce à Multipass, nous avons créé deux machines virtuelles Ubuntu indépendantes et les avons reliées dans un même réseau virtuel.
Nous avons vérifié leur communication à l'aide de la commande `ping`, installé un serveur web Nginx, puis mis en place des règles de sécurité avec le pare-feu UFW.
Enfin, nous avons observé les ressources matérielles consommées par la machine virtuelle afin de mieux comprendre la gestion des infrastructures dans un environnement cloud.

Ce TP montre qu'une infrastructure similaire à celle proposée par les fournisseurs cloud peut être reproduite localement à l'aide de machines virtuelles légères, tout en conservant les mêmes principes de déploiement, de publication de services et de sécurisation du réseau.