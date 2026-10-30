Compte Rendu - TP 2 : Premiers conteneurs avec Docker

Trinôme : DOHOU DEBORAH ELIANE & KRI BANTI HERMES & CISSE FATIM 
Cours : Introduction au Cloud Computing (Licence 3 - RIST / BDGL)  
Enseignant : BAKAYOKO Ibrahima  




Partie 0 — Préparer la VM et installer Dock

 Commandes exécutées et sorties du terminal
```bash
mario@mario-HP-EliteBook-840-G6:~\$ sudo apt update && sudo apt install -y docker.io docker-buildx docker-compose-v2
 [...]
 docker.io est déjà la version la plus récente (29.1.3-0ubuntu3~22.04.2).
 Les NOUVEAUX paquets suivants seront installés : docker-buildx docker-compose-v2
 Paramétrage de docker-compose-v2 (2.40.3+ds1-0ubuntu1~22.04.1) ...
 Paramétrage de docker-buildx (0.30.1-0ubuntu1~22.04.1) ...

mario@mario-HP-EliteBook-840-G6:~ sudo usermod -aG docker USER
mario@mario-HP-EliteBook-840-G6:~\$ newgrp docker

mario@mario-HP-EliteBook-840-G6:~\$ docker version
Client:
 Version:           29.1.3
 API version:       1.52
 Go version:        go1.24.4
 Git commit:        29.1.3-0ubuntu3~22.04.2
 Built:             Wed Apr 29 22:18:59 2026
 OS/Arch:           linux/amd64
 Context:           default

Server:
 Engine:
  Version:          29.1.3
  API version:      1.52 (minimum version 1.44)
  Go version:       go1.24.4
  Git commit:       29.1.3-0ubuntu3~22.04.2
  Built:            Wed Apr 29 22:18:59 2026
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          2.2.1
 runc:
  Version:          1.3.4-0ubuntu1~22.04.1
 docker-init:
  Version:          0.19.0

mario@mario-HP-EliteBook-840-G6:~\$ docker info | grep -E "Server Version|Storage Driver|Cgroup|Kernel Version"
 Server Version: 29.1.3
 Storage Driver: overlayfs
 Cgroup Driver: systemd
 Cgroup Version: 2
 Kernel Version: 6.8.0-138-generic
```

 Réponses aux questions

Q1. Quelle version de Docker est installée ? Quel noyau Docker utilise-t-il (ligne Kernel Version) ?
    Version de Docker installée : La version installée est la 29.1.3.
    Noyau Linux utilisé par Docker : Docker utilise le noyau  6.8.0-138-generic .

Q2. Être membre du groupe docker revient presque à être administrateur de la machine. Pourquoi ? Quelle précaution cela impose-t-il sur un serveur partagé ?

 Pourquoi : Le démon Docker s'exécute avec les privilèges root. Un utilisateur appartenant au groupe docker peut lancer un conteneur en lui donnant accès à la totalité du disque de la machine hôte (avec l'option -v /:/mnt par exemple). Il devient alors possible de modifier n'importe quel fichier du système hôte (comme /etc/shadow), contournant ainsi toutes les restrictions de sécurité.
Précaution requise : Sur un serveur partagé, il ne faut jamais ajouter d'utilisateurs non administrateurs ou sans vérification stricte dans le groupe docker. L'accès à Docker doit être traité avec la même rigueur et les mêmes audits que l'accès au privilège sudo.





Partie 1 — Lancer et inspecter des conteneurs

 1.1 Premier conteneur
```bash
mario@mario-HP-EliteBook-840-G6:~\$ docker run hello-world

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

mario@mario-HP-EliteBook-840-G6:~\$ docker images
IMAGE                     ID             DISK USAGE   CONTENT SIZE   EXTRA
debodev/apifilrouge:1.0   e846e55cf26c        198MB         48.3MB        
hello-world:latest        11a3eb1597e8       25.9kB         9.49kB    U   
mario/apifilrouge:1.0     e846e55cf26c        198MB         48.3MB        
nginx:latest              4c41e2d0a3a5        243MB         66.8MB    U   
python:3.12-slim          a0f57064cbdc        179MB         45.4MB        

mario@mario-HP-EliteBook-840-G6:~\$ docker ps -a
CONTAINER ID   IMAGE         COMMAND                  CREATED          STATUS                              PORTS                                     NAMES
a4278375cb92   hello-world   "/hello"                 1 second ago     Exited (0) Less than a second ago                                             hungry_shamir
bbf7c1147266   nginx         "/docker-entrypoint.…"   25 minutes ago   Up 25 minutes                       0.0.0.0:8080->80/tcp, [::]:8080->80/tcp   web
```

 1.2 Un serveur web en une commande
```bash
mario@mario-HP-EliteBook-840-G6:~\$ docker run -d --name web -p 8080:80 nginx
docker ps
curl -s http://localhost:8080 | head -n 5
docker: Error response from daemon: Conflict. The container name "/web" is already in use by container "bbf7c1147266ee45aef3409d15934f2b7995ff97983a17f3620e91ec4e80110b".

mario@mario-HP-EliteBook-840-G6:~\$ docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS          PORTS                                     NAMES
bbf7c1147266   nginx     "/docker-entrypoint.…"   29 minutes ago   Up 29 minutes   0.0.0.0:8080->80/tcp, [::]:8080->80/tcp   web
```

1.3 Regarder à l'intérieur
```bash
mario@mario-HP-EliteBook-840-G6:~\$ docker exec web cat /etc/os-release | head -n 2
PRETTY_NAME="Debian GNU/Linux 13 (trixie)"
NAME="Debian GNU/Linux"

mario@mario-HP-EliteBook-840-G6:~\$ cat /etc/os-release | head -n 2
PRETTY_NAME="Ubuntu 22.04.5 LTS"
NAME="Ubuntu"

mario@mario-HP-EliteBook-840-G6:~\$ docker exec web uname -r
6.8.0-138-generic

mario@mario-HP-EliteBook-840-G6:~\$ uname -r
6.8.0-138-generic

mario@mario-HP-EliteBook-840-G6:~\$ docker logs --tail 5 web
2026/10/25 00:29:56 [notice] 1#1: start worker process 34
2026/10/25 00:29:56 [notice] 1#1: start worker process 35
2026/10/25 00:29:56 [notice] 1#1: start worker process 36
172.17.0.1 - - [25/Oct/2026:00:30:09 +0000] "GET / HTTP/1.1" 200 58 "-" "curl/7.81.0" "-"
172.17.0.1 - - [25/Oct/2026:00:58:59 +0000] "GET / HTTP/1.1" 200 58 "-" "curl/7.81.0" "-"

mario@mario-HP-EliteBook-840-G6:~\$ docker top web
UID                 PID                 PPID                C                   STIME               TTY                 TIME                CMD
root                90832               90809               0                   00:29               ?                   00:00:00            nginx: master process nginx -g daemon off;
systemd+            90909               90832               0                   00:29               ?                   00:00:00            nginx: worker process

mario@mario-HP-EliteBook-840-G6:~\$ ps aux | grep "nginx: master" | grep -v grep
root       90832  0.0  0.1  14968  8768 ?        Ss   00:29   0:00 nginx: master process nginx -g daemon off;

mario@mario-HP-EliteBook-840-G6:~\$ docker stats --no-stream web
CONTAINER ID   NAME      CPU %     MEM USAGE / LIMIT     MEM %     NET I/O           BLOCK I/O       PIDS
bbf7c1147266   web       0.00%     7.609MiB / 7.567GiB   0.10%     6.42kB / 1.56kB   86kB / 12.3kB   9
```

1.4 Un conteneur est éphémère

Tentative de modification sans volume persistent :
```bash
mario@mario-HP-EliteBook-840-G6:~\$ docker exec web sh -c 'echo "<h1>Binôme Mario - Deborah</h1>" > /usr/share/nginx/html/index.html'
 Si le conteneur d'origine tourne sans volume, cette modification fonctionne. 
 Si un volume en lecture seule (:ro) est déjà en place, le système renvoie :
sh: 1: cannot create /usr/share/nginx/html/index.html: Read-only file system
```

 Recréation du conteneur en utilisant un volume persistant de l'hôte :
```bash
mario@mario-HP-EliteBook-840-G6:~\$ mkdir -p ~/site
mario@mario-HP-EliteBook-840-G6:~\$ echo "<h1>Binôme Mario - Deborah : servi depuis un volume</h1>" > ~/site/index.html

mario@mario-HP-EliteBook-840-G6:~\$ docker rm -f web
web

mario@mario-HP-EliteBook-840-G6:~\$ docker run -d --name web -p 8080:80 -v ~/site:/usr/share/nginx/html:ro nginx
078d5af53c34fd59d20e1c69b9c13098971b4352703bf316f18fbaa3d8722712

mario@mario-HP-EliteBook-840-G6:~\$ curl -s http://localhost:8080
<h1>Binôme Mario - Deborah : servi depuis un volume</h1>
```

 Réponses aux questions

Q1. D'après le message de hello-world, quelles sont les quatre étapes réalisées par Docker ?**
    1. Le client Docker contacte le démon Docker.
    2. Le démon Docker télécharge (pull) l'image "hello-world" depuis le Docker Hub car elle n'existe pas localement.
    3. Le démon Docker crée un nouveau conteneur basé sur cette image et lance l'exécutable textuel.
    4. Le démon Docker redirige le flux de sortie vers le client Docker, qui l'affiche sur le terminal de l'utilisateur.

Q2. Le conteneur web et la VM ont-ils la même distribution ? Le même noyau ? Expliquez à l'aide du cours.
    Distribution : Non. Le conteneur web s'exécute sur une distribution  Debian GNU/Linux 13 (trixie) , alors que l'hôte s'exécute sur  Ubuntu 22.04.5 LTS .
    Noyau : Oui, les deux affichent la version exacte 6.8.0-138-generic . Un conteneur n'embarque pas son propre noyau, il isole ses processus en utilisant directement le noyau Linux de la machine hôte.

Q3. Les processus Nginx du conteneur apparaissent-ils dans ps aux sur la VM ? Qu'est-ce que cela montre sur la nature d'un conteneur ?
  Réponse : Oui. Le processus d'ID `90832` apparaît à la fois dans le docker top web et dans le ps aux du système Ubuntu. Cela démontre qu'un conteneur n'est pas une machine virtuelle isolée par un hyperviseur, mais un simple groupe de processus de l'hôte isolés par des namespaces (espaces de noms) et limités par des cgroups au sein du même noyau.

Q4. Retracez le chemin d'une requête du navigateur de l'hôte jusqu'à Nginx : quelles redirections de port traverse-t-elle ?
      La requête émise depuis la machine hôte vers l'adresse de bouclage 127.0.0.1:8080 est d'abord captée par l'interface réseau locale.
     Elle franchit la redirection de port gérée par le moteur Docker (`0.0.0.0:8080->80/tcp`), qui traduit et achemine le trafic du port `8080` de l'hôte vers le port interne `80` affecté à l'intérieur du conteneur Nginx.

Q5. Pourquoi la page modifiée a-t-elle disparu après docker rm ? Que change l'option -v ?
    Disparition : Sans volume, les écritures d'un conteneur s'effectuent dans sa couche supérieure éphémère. Lors de l'exécution de docker rm, cette couche est définitivement détruite avec toutes les modifications.
Option -v : L'option -v configure un point de montage qui externalise et lie de manière persistante un répertoire du système hôte (`~/site`) vers un dossier interne du conteneur (`/usr/share/nginx/html`). Les fichiers résident sur le disque physique de l'hôte et survivent aux cycles de suppression des conteneurs.






 Partie 2 — Comparer conteneur et VM

Commandes exécutées et sorties du terminal
```bash
mario@mario-HP-EliteBook-840-G6:~\$ docker pull alpine
Using default tag: latest
latest: Pulling from library/alpine
7b229517ed22: Pull complete 
3da1513d2124: Download complete 
3ec36f3670df: Download complete 
Digest: sha256:be2656061b536c1883c4034790e21d0a09c5aaaf51968b89368a3c0371ea2c9a
Status: Downloaded newer image for alpine:latest
docker.io/library/alpine:latest

mario@mario-HP-EliteBook-840-G6:~\$ docker images
IMAGE                     ID             DISK USAGE   CONTENT SIZE   EXTRA
alpine:latest             be2656061b53         13MB         3.94MB        
debodev/apifilrouge:1.0   e846e55cf26c        198MB         48.3MB        
hello-world:latest        11a3eb1597e8       25.9kB         9.49kB    U   
mario/apifilrouge:1.0     e846e55cf26c        198MB         48.3MB        
nginx:latest              4c41e2d0a3a5        243MB         66.8MB    U   
python:3.12-slim          a0f57064cbdc        179MB         45.4MB        

mario@mario-HP-EliteBook-840-G6:~\$ time docker run --rm alpine echo "Bonjour depuis un conteneur"
Bonjour depuis un conteneur

real	0m0,296s
user	0m0,005s
sys	0m0,012s

mario@mario-HP-EliteBook-840-G6:~\$ time docker run --rm alpine echo "Bonjour depuis un conteneur"
Bonjour depuis un conteneur

real	0m0,251s
user	0m0,007s
sys	0m0,012s

mario@mario-HP-EliteBook-840-G6:~\$ systemd-analyze
Startup finished in 13.248s (firmware) + 7.454s (loader) + 2.631s (kernel) + 10.455s (userspace) = 33.789s 
graphical.target reached after 10.437s in userspace

mario@mario-HP-EliteBook-840-G6:~\$ free -h
               total       utilisé      libre     partagé tamp/cache   disponible
Mem:           7,6Gi       2,8Gi       835Mi       1,0Gi       4,0Gi       3,5Gi
Échange:      4,8Gi       697Mi       4,1Gi
```

Tableau comparatif des mesures

Élément
Système d'exploitation
Taille sur disque
Temps de démarrage
Mémoire utilisée
VM ubuntu-srv
Ubuntu 22.04 LTS complet
~2.1 GiB (relevé TP1)
33.789s (systemd-analyze)
2.8 GiB (colonne utilisé)
Conteneur nginx
Debian GNU/Linux 13
243 MB
< 0.5s (Instantané)
7.609 MiB (docker stats)
Conteneur alpine
Alpine Linux minime
13 MB
0.251s (time docker run)
Éphémère (~0 MiB après run)


 Réponses aux questions

Q1. Dans quel rapport (environ ×10, ×100…) le conteneur est-il plus léger et plus rapide que la VM ? Vos mesures confirment-elles le tableau du cours ?
    Rapport de légèreté : L'image du conteneur `alpine` (13 MB) est environ 160 fois plus légère sur disque que le système d'exploitation complet de la VM (~2.1 GB).
    Rapport de rapidité : Le conteneur démarre et s'exécute en 0.251s , ce qui est environ 134 fois plus rapide que le temps d'initialisation complet de la VM (33.789s). 

    Conclusion : Ces mesures empiriques confirment parfaitement le tableau du cours qui souligne que la virtualisation par conteneur élimine la surcharge d'un hyperviseur et d'un OS invité complet.

Q2. Pourquoi le premier time docker run est-il plus long que le second ?
 Réponse : Lors de la première exécution (`0.296s`), Docker doit instancier pour la première fois les espaces de noms (namespaces), allouer les interfaces de pont réseau virtuels et charger les métadonnées de l'image sur le disque. Lors du second appel (`0.251s`), ces composants d'isolation ainsi que les structures du système de fichiers sont déjà en cache chaud dans la mémoire du noyau Linux, ce qui réduit le temps de traitement.

Q3. Citez un cas où vous choisiriez quand même une VM plutôt qu'un conteneur.
Réponse : On choisit une machine virtuelle lorsqu'on doit exécuter des applications nécessitant un noyau de système d'exploitation différent de celui de l'hôte (par exemple, faire tourner un serveur Windows ou un BSD sur un serveur physique Linux). Les VM sont également indispensables lorsque l'on souhaite une isolation de sécurité matérielle stricte ou une gestion personnalisée des couches de pilotes de périphériques bas niveau.






 Partie 3 — Conteneuriser l'API fil rouge

 3.1 Création des fichiers et construction de l'image (v1.0)
Les fichiers de l'application Python (`app.py`), des dépendances (`reqirements.txt`), de configuration Docker (`Dockerfile`) et d'exclusion (`.dockerignore`) ont été correctement initialisés au sein du répertoire `~/api`.

```bash
mario@mario-HP-EliteBook-840-G6:~\$ mkdir -p ~/api && cd ~/api

mario@mario-HP-EliteBook-840-G6:~/api\$ docker build -t api:1.0 .
[+] Building 12.5s (10/10) FINISHED                              docker:default
 => [internal] load build definition from Dockerfile                       0.1s
 => => transferring dockerfile: 195B                                       0.0s
 => [internal] load metadata for docker.io/library/python:3.12-slim        0.0s
 => [internal] load .dockerignore                                          0.0s
 => => transferring context: 64B                                           0.0s
 => [internal] load build context                                          0.0s
 => => transferring context: 752B                                          0.0s
 => [1/5] FROM docker.io/library/python:3.12-slim@sha256:a0f57064cbdcd628  0.1s
 => => resolve docker.io/library/python:3.12-slim@sha256:a0f57064cbdcd628  0.0s
 => [2/5] WORKDIR /app                                                     0.0s
 => [3/5] COPY requirements.txt .                                          0.0s
 => [4/5] RUN pip install --no-cache-dir -r requirements.txt              11.2s
 => [5/5] COPY . .                                                         0.0s
 => exporting to image                                                     0.9s
 => => exporting layers                                                    0.6s
 => => exporting manifest sha256:e635c7141ee7dda1657b8e46edb64f450b6f30a0  0.0s
 => => exporting config sha256:f4ef388dc5cb4062516d503d643cff02bb1b3bc4f8  0.0s
 => => exporting attestation manifest sha256:952e8908f9d1e82f1580f37313a5  0.0s
 => => exporting manifest list sha256:5b204c6332cf497c45181c0f1b59cdf4cb2  0.0s
 => => naming to docker.io/library/api:1.0                                 0.0s
 => => unpacking to docker.io/library/api:1.0                              0.2s

mario@mario-HP-EliteBook-840-G6:~/api\$ docker images api
IMAGE     ID             DISK USAGE   CONTENT SIZE   EXTRA
api:1.0   5b204c6332cf        198MB         48.3MB        

mario@mario-HP-EliteBook-840-G6:~/api\$ docker history api:1.0
IMAGE          CREATED          CREATED BY                                      SIZE      COMMENT
5b204c6332cf   1 second ago     CMD ["python" "app.py"]                         0B        buildkit.dockerfile.v0
<missing>      1 second ago     EXPOSE [5000/tcp]                               0B        buildkit.dockerfile.v0
<missing>      1 second ago     COPY . . # buildkit                             20.5kB    buildkit.dockerfile.v0
<missing>      1 second ago     RUN /bin/sh -c pip install --no-cache-dir -r…   15.3MB    buildkit.dockerfile.v0
<missing>      12 seconds ago   COPY requirements.txt . # buildkit              12.3kB    buildkit.dockerfile.v0
<missing>      12 seconds ago   WORKDIR /app                                    8.19kB    buildkit.dockerfile.v0
<missing>      4 days ago       CMD ["python3"]                                 0B        buildkit.dockerfile.v0
<missing>      4 days ago       RUN /bin/sh -c set -eux;  for src in idle3 p…   16.4kB    buildkit.dockerfile.v0
<missing>      4 days ago       RUN /bin/sh -c set -eux;   savedAptMark="$(a…   41.4MB    buildkit.dockerfile.v0
<missing>      4 days ago       ENV PYTHON_SHA256=c2c4321961fab0fb999d66e0ce…   0B        buildkit.dockerfile.v0
<missing>      4 days ago       ENV PYTHON_VERSION=3.12.15                      0B        buildkit.dockerfile.v0
<missing>      4 days ago       ENV GPG_KEY=7169605F62C751356D054A26A821E680…   0B        buildkit.dockerfile.v0
<missing>      4 days ago       RUN /bin/sh -c set -eux;  apt-get update;  a…   4.93MB    buildkit.dockerfile.v0
<missing>      4 days ago       ENV LANG=C.UTF-8                                0B        buildkit.dockerfile.v0
<missing>      4 days ago       ENV PATH=/usr/local/bin:/usr/local/sbin:/usr…   0B        buildkit.dockerfile.v0
<missing>      4 days ago       # debian.sh --arch 'amd64' out/ 'trixie' '@1…   87.7MB    debuerreotype 0.17
```

3.2 Exécution de l'API et mise à l'échelle horizontale (Scale-out)
```bash
mario@mario-HP-EliteBook-840-G6:~/api\$ docker run -d --name api -p 5000:5000 -e BINOME="Mario-Deborah" api:1.0
35169ec75f2daa0cbbe98c083fcbb6ae6bec460b6ee98d1e15c4a056689a28d7

mario@mario-HP-EliteBook-840-G6:~/api\$ for p in 5001 5002 5003; do docker run -d --name api-p -p p:5000 -e BINOME="Mario-Deborah" api:1.0; done
ba71a9dd04fc81da4d8f263b920804423b6a6c9141ebe31c966a67baa3de2531
caa259cb8c327a1d4469f89ef302defd064e74444eebcb4a0e75590e0921cbca
53b12b514e982dbb777a001eadb63c4b1516388a656859f0d8b16ea1bbcd136f

mario@mario-HP-EliteBook-840-G6:~/api\$ for p in 5001 5002 5003; do curl -s http://localhost:p; echo; done
{"binome":"Mario-Deborah","conteneur":"ba71a9dd04fc","message":"API fil rouge"}
{"binome":"Mario-Deborah","conteneur":"caa259cb8c32","message":"API fil rouge"}
{"binome":"Mario-Deborah","conteneur":"53b12b514e98","message":"API fil rouge"}

mario@mario-HP-EliteBook-840-G6:~/api\$ docker rm -f api-5001 api-5002 api-5003
api-5001
api-5002
api-5003
```

 3.3 Ajout de la route /version et observation du cache (v1.1)
```bash
mario@mario-HP-EliteBook-840-G6:~/api\$ docker build -t api:1.1 .
[+] Building 0.3s (10/10) FINISHED                               docker:default
 => [internal] load build definition from Dockerfile                       0.0s
 => => transferring dockerfile: 195B                                       0.0s
 => [internal] load metadata for docker.io/library/python:3.12-slim        0.0s
 => [internal] load .dockerignore                                          0.0s
 => => transferring context: 64B                                           0.0s
 => [1/5] FROM docker.io/library/python:3.12-slim@sha256:a0f57064cbdcd628  0.0s
 => => resolve docker.io/library/python:3.12-slim@sha256:a0f57064cbdcd628  0.0s
 => [internal] load build context                                          0.0s
 => => transferring context: 604B                                          0.0s
 => CACHED [2/5] WORKDIR /app                                              0.0s
 => CACHED [3/5] COPY requirements.txt .                                   0.0s
 => CACHED [4/5] RUN pip install --no-cache-dir -r requirements.txt        0.0s
 => [5/5] COPY . .                                                         0.0s
 => exporting to image                                                     0.1s
 => => exporting layers                                                    0.0s
 => => exporting manifest sha256:b062f830aade9a204b47141e71c4df5e0561cb7a  0.0s
 => => exporting config sha256:30d878e507216570010cd884ee3b6c79dad126a14e  0.0s
 => => exporting attestation manifest sha256:27706471c21be68717bfc2dbabfa  0.0s
 => => exporting manifest list sha256:22b0f5a303f7c069979a4b458d2d4b6f819  0.0s
 => => naming to docker.io/library/api:1.1                                 0.0s
 => => unpacking to docker.io/library/api:1.1                              0.0s
```

Réponses aux questions

Q1. Quelle est la taille de l'image api:1.0 ? Combien de couches docker history affiche-t-il, et lesquelles viennent de votre Dockerfile ?
    Taille de l'image : L'image `api:1.0` possède une taille globale virtuelle sur disque de 198 MB (pour un contenu applicatif compressé net de 48.3 MB).
    Nombre de couches :La commande `docker history` affiche 15 couches au total.
    Couches venant de notre Dockerfile : Les 6 couches supérieures  (allant du point d'entrée `CMD` tout en haut jusqu'à la création du dossier de travail `WORKDIR /app`) proviennent directement des instructions de notre Dockerfile. Les couches inférieures marquées `<missing>` proviennent de la structure parente officielle de `python:3.12-slim`.

*   **Q2. Pourquoi copie-t-on requirements.txt et installe-t-on les dépendances avant de copier le code ? Appuyez-vous sur les lignes CACHED de l'étape 6.**
    *   **Réponse :** Docker met en cache le résultat de chaque étape d'un build. Si les fichiers ciblés par une étape n'ont pas été modifiés, Docker réutilise instantanément la couche existante sans rien recalculer, ce qui est indiqué par la mention `CACHED`. Étant donné que le code applicatif (`app.py`) change très souvent mais que les bibliothèques requises (`requirements.txt`) évoluent rarement, séparer ces opérations permet d'isoler l'étape lourde de téléchargement réseau `RUN pip install`. Les lignes `CACHED [2/5]`, `CACHED [3/5]` et `CACHED [4/5]` observées lors du passage à la version 1.1 prouvent que Docker a ignoré l'installation de Flask, réduisant le temps de build à seulement 0,3 seconde.

Q3. Que renvoie le champ conteneur pour les trois copies de l'étape 5 ? Quelle caractéristique du cloud (cours, séance 1) cette expérience illustre-t-elle ?
    Ce que renvoie le champ conteneur : Le champ `conteneur` renvoie l'identifiant hexadécimal court et unique de la machine virtuelle isolée qui exécute l'application (`ba71a9dd04fc`, `caa259cb8c32` et `53b12b514e98`).
  Caractéristique du Cloud illustrée : Cette manipulation illustre la  mise à l'échelle horizontale (Elasticity / Scale-Out) . Elle démontre la capacité du Cloud à instancier instantanément de multiples serveurs parfaitement identiques à partir d'une unique image de conteneur immuable pour répartir la charge de trafic utilisateur.
Q4. Pourquoi passer le nom du binôme par une variable d'environnement (-e) plutôt que de l'écrire dans le code ? Quel lien avec les secrets (mots de passe, clés) ?  Pourquoi : Cela permet de décorréler la configuration spécifique d'un environnement de la logique pure du code de l'application... 
Lien avec les secrets : C'est une recommandation de sécurité absolue (méthodologie des Twelve-Factor Apps)…








Partie 4 — Publier l'image et nettoyer

### Commandes exécutées et sorties du terminal
```bash
mario@mario-HP-EliteBook-840-G6:~/api\$ docker login
Authenticating with existing credentials... [Username: debodev]
Login Succeeded

mario@mario-HP-EliteBook-840-G6:~/api\$ docker tag api:1.1 debodev/api-fil-rouge:1.1

mario@mario-HP-EliteBook-840-G6:~/api\$ docker push debodev/api-fil-rouge:1.1
The push refers to repository [docker.io/debodev/api-fil-rouge]
f259963da058: Waiting 
a1f4af254f5b: Pushed 
45e4b3f25472: Pushed 
442d9c892c75: Pushed 
ca533669db97: Pushed 
1.1: digest: sha256:22b0f5a303f7c069979a4b458d2d4b6f8193248a463167dfc3531237ca1e7374 size: 856

mario@mario-HP-EliteBook-840-G6:~/api docker rm -f (docker ps -aq)
35169ec75f2d
078d5af53c34
a4278375cb92

mario@mario-HP-EliteBook-840-G6:~/api\$ docker system df
TYPE            TOTAL     ACTIVE    SIZE      RECLAIMABLE
Images          7         0         374.8MB   474.9kB (0%)
Containers      0         0         0B        0B
Local Volumes   0         0         0B        0B
Build Cache     12        0         63.94MB   20.66MB

mario@mario-HP-EliteBook-840-G6:~/api\$ docker image prune -a -f
Deleted Images:
untagged: alpine:latest
deleted: sha256:be2656061b536c1883c4034790e21d0a09c5aaaf51968b89368a3c0371ea2c9a
untagged: api:1.1
untagged: api:1.0
deleted: sha256:5b204c6332cf497c45181c0f1b59cdf4cb277932e1c70a84fb851e655c0e6a00
untagged: hello-world:latest
deleted: sha256:11a3eb1597e864c51b3f56733cb2e8a90bf0c284bfc05f9444d81c558bc848aa
untagged: nginx:latest
deleted: sha256:4c41e2d0a3a55b325ed572cd0200bdb72af1433c3daa2bca6ef8eea68ce504a2
untagged: python:3.12-slim
untagged: debodev/api-fil-rouge:1.1
untagged: debodev/apifilrouge:1.0
deleted: sha256:e846e55cf26cdb1b05937b6a46e777079c42ff25f18d2cd149f29fdcb896d9a5

Total reclaimed space: 48.13MB

mario@mario-HP-EliteBook-840-G6:~/api\$ docker system df
TYPE            TOTAL     ACTIVE    SIZE      RECLAIMABLE
Images          0         0         149.7MB   149.7MB (100%)
Containers      0         0         0B        0B
Local Volumes   0         0         0B        0B
Build Cache     12        0         63.94MB   63.94MB
```

 Réponses aux questions de la Partie 4

Q1. Indiquez l'adresse complète de votre image publiée. Est-elle publique ou privée ?
    Adresse de l'image : `docker.io/debodev/api-fil-rouge:1.1`
    Statut : L'image est  publique . N'importe quel serveur ou membre de l'équipe peut la télécharger directement à l'aide de la commande `docker pull debodev/api-fil-rouge:1.1`.

Q2. Pourquoi étiqueter l'image avec un numéro de version (1.1) plutôt que de toujours utiliser latest ?
    Réponse : Utiliser des identifiants numériques fixes apporte des garanties d' immutabilité et de traçabilité. L'étiquette `latest` est un pointeur changeant qui se met automatiquement à jour lors de chaque nouvelle soumission. S'appuyer sur `latest` en production présente un risque critique : deux machines démarrant le même conteneur à quelques jours d'intervalle pourraient se retrouver avec des versions de code différentes, ce qui rend l'identification des anomalies impossible.

Q3. Quel espace disque docker image prune a-t-il libéré ? Dans le cloud, pourquoi faut-il aussi nettoyer les registres ?
    Espace libéré :La commande de purge locale a libéré un volume de cache immédiat de 48.13 MB en supprimant les couches mortes accumulées, laissant un espace d'images inutilisées entièrement récupérable à 100%.
    Nettoyage dans le cloud :Les fournisseurs d'infrastructures Cloud facturent le stockage physique des données au gigaoctet mensuel. Laisser des centaines d'images intermédiaires ou obsolètes de builds s'accumuler inutilement au fil des déploiements automatiques alourdit directement les coûts de stockage de l'entreprise.
