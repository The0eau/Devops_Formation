# Résumé de la formation Docker : Exécuter un conteneur

Ce document résume les étapes de base pour exécuter et interagir avec des conteneurs Docker.

## 1. Exécuter un conteneur

Pour exécuter votre premier conteneur, vous pouvez utiliser l'interface de ligne de commande (CLI) de Docker.

Ouvrez un terminal et exécutez la commande suivante :
```bash
$ docker container run -t ubuntu top
```

- **`docker container run`**: Commande pour démarrer un nouveau conteneur.
- **`-t`**: Alloue un pseudo-TTY, nécessaire pour des commandes interactives comme `top`.
- **`ubuntu`**: L'image à partir de laquelle créer le conteneur. Si elle n'est pas disponible localement, Docker la téléchargera (`docker pull`) depuis le Docker Hub.
- **`top`**: La commande à exécuter à l'intérieur du conteneur.

`top` est un utilitaire Linux qui affiche les processus en cours. Dans le conteneur, vous remarquerez qu'il n'y a que le processus `top` lui-même. C'est un exemple de l' **isolation du namespace PID**, qui isole les identifiants de processus du conteneur de ceux de l'hôte.

**Note importante :** Le conteneur utilise le noyau de l'hôte. L'image (ici, `ubuntu`) fournit uniquement le système de fichiers et les outils.

## 2. Interagir avec un conteneur en cours d'exécution

Pour interagir avec un conteneur déjà en marche, vous devez d'abord obtenir son ID.

1.  Ouvrez un **nouveau terminal**.
2.  Listez les conteneurs en cours d'exécution :
    ```bash
    $ docker container ls
    ```
3.  Utilisez l'ID du conteneur pour exécuter une commande à l'intérieur avec `docker container exec`. Pour ouvrir un shell `bash` interactif, utilisez :
    ```bash
    $ docker container exec -it <ID_DU_CONTENEUR> bash 
    root@<ID_DU_CONTENEUR>:/#
    ```
    - **`-it`**: Combine `-i` (mode interactif) et `-t` (pseudo-TTY) pour interagir avec le shell.

Le prompt de votre terminal changera, indiquant que vous êtes maintenant à l'intérieur du conteneur. Cette méthode ne nécessite pas de serveur SSH.

Une fois à l'intérieur, vous pouvez inspecter les processus du conteneur :
```bash
$ ps -ef
```
Vous ne verrez que les processus isolés dans le namespace du conteneur (`top`, `bash`, `ps`).

## 3. Les Namespaces Linux

Les conteneurs utilisent les namespaces du noyau Linux pour l'isolation des ressources. En plus du namespace PID, il existe :

- **`MNT`**: Isole les points de montage.
- **`NET`**: Fournit une pile réseau isolée pour chaque conteneur.
- **`IPC`**: Isole la communication inter-processus.
- **`Utilisateur`**: Isole les identifiants d'utilisateurs.
- **`UTC`**: Permet de définir un nom d'hôte et de domaine spécifique au conteneur.

## 4. Nettoyage

1.  Pour quitter le shell du conteneur :
    ```bash
    root@<ID_DU_CONTENEUR>:/# exit
    ```
2.  Pour arrêter le conteneur exécutant `top`, retournez au premier terminal et appuyez sur `<ctrl>-c`.

## 5. Notes sur la compatibilité multiplateforme

- **Docker sur Windows et Mac**: Docker utilise un sous-système Linux intégré (basé sur le projet open-source `LinuxKit`) pour exécuter des conteneurs Linux sur ces systèmes.
- **Conteneurs Windows natifs**: Il est également possible d'exécuter des conteneurs natifs sur Windows 10 et Windows Server 2016 (ou versions ultérieures).

---

# Part 2 : Utilisation d'images du Docker Hub

*Il est recommandé d'utiliser l'édition Docker Personal pour cette pratique.*

## 1. Explorer le Docker Hub

Le **Docker Hub** est le registre public central pour les images Docker. On y trouve :
- **Images communautaires** : Partagées publiquement par n'importe qui.
- **Images officielles/vérifiées** : Contenu vérifié et scanné pour des vulnérabilités par Docker. Les images certifiées sont prêtes pour l'entreprise.

**Important** : Pour la production, évitez d'utiliser du contenu non vérifié, car il peut contenir des failles de sécurité ou des logiciels malveillants.

## 2. Exécuter des services en conteneurs

### Serveur Web NGINX

Exécutez un serveur NGINX en utilisant l'image officielle :
```bash
$ docker container run --detach --publish 8080:80 --name nginx nginx
```
Nouveaux drapeaux :
- **`--detach`**: Fait tourner le conteneur en arrière-plan.
- **`--publish 8080:80`**: Mappe le port `8080` de l'hôte au port `80` du conteneur.
- **`--name nginx`**: Nomme le conteneur `nginx` pour une référence plus facile.

Accédez au serveur NGINX via votre navigateur à l'adresse **http://localhost:8080**.

### Base de données MongoDB

Exécutez un serveur MongoDB en spécifiant une version :
```bash
$ docker container run --detach --publish 8081:27017 --name mongo mongo:3.4
```
- **`mongo:3.4`**: Utilise la version `3.4` de l'image Mongo. Par défaut, le tag `latest` est utilisé.
- **`--publish 8081:27017`**: Mappe le port `8081` de l'hôte au port par défaut de Mongo, `27017`. Le port de l'hôte doit être unique.

Accédez à **http://localhost:8081** pour voir une sortie de Mongo.

## 3. Vérifier les conteneurs

Pour lister les conteneurs en cours d'exécution :
```bash
$ docker container ls
```
Vous verrez les conteneurs `nginx` et `mongo` avec leurs noms et les mappages de ports.

La colonne `COMMAND` peut montrer une commande comme `docker-entrypoint`. Il s'agit d'un script de configuration qui s'exécute au démarrage du conteneur avant le processus principal. Le code source de ces scripts est généralement disponible sur GitHub, lié depuis la page de l'image sur le Docker Hub.

## 4. Les avantages de l'isolation

L'isolation des conteneurs permet de :
- **Éviter les conflits de dépendances** : Exécutez des applications avec des versions différentes de la même dépendance (ex: Java 7 et Java 8) sur le même hôte.
- **Exécuter plusieurs services identiques** : Faites tourner plusieurs conteneurs NGINX qui écoutent tous sur le port `80` (à l'intérieur de leur propre réseau), en les mappant à des ports uniques sur l'hôte.
- **Garder l'hôte propre** : Les dépendances sont incluses dans l'image, il n'est donc pas nécessaire de les installer sur la machine hôte.

Cela est rendu possible par les **namespaces Linux**.

L'exécution de plusieurs conteneurs sur un seul hôte optimise l'utilisation des ressources (CPU, mémoire) et peut entraîner des économies de coûts significatives.

---

# Part 3 : Supprimer les conteneurs

Après avoir expérimenté, il est important de nettoyer votre environnement.

## 1. Arrêter les conteneurs

1.  Obtenez une liste des conteneurs en cours d'exécution :
    ```bash
    $ docker container ls
    ```
2.  Arrêtez les conteneurs en utilisant leur ID ou leur nom. Vous pouvez arrêter plusieurs conteneurs en une seule commande.
    ```bash
    # Par ID ou nom de conteneur
    $ docker container stop <ID_ou_Nom_1> <ID_ou_Nom_2>
    ```
    **Astuce** : Quelques caractères de l'ID sont généralement suffisants pour identifier un conteneur de manière unique.

## 2. Supprimer les ressources non utilisées

Une fois les conteneurs arrêtés, vous pouvez les supprimer et nettoyer d'autres ressources (réseaux, volumes, images) qui ne sont plus utilisées.
```bash
$ docker system prune
```
Cette commande supprime :
- Tous les conteneurs arrêtés.
- Tous les réseaux non utilisés par au moins un conteneur.
- Tous les volumes non utilisés par au moins un conteneur.
- Toutes les images "dangling" (images sans tag, qui ne sont parentes d'aucune image taguée).

---

# Part 4 : Créer une application Python simple

## 1. Créer le fichier de l'application

Copiez et collez cette commande entière dans votre terminal pour créer un fichier `app.py`.

```bash
echo 'from flask import Flask

app = Flask(__name__)

@app.route("/")
def hello():
    return "hello world!"

if __name__ == "__main__":
    app.run(host="0.0.0.0")' > app.py
```
Ceci est une application Python simple qui utilise le framework Flask pour exposer un serveur web HTTP sur le port 5000 (le port par défaut de Flask).

## 2. (Optionnel) Exécuter l'application localement

Si vous avez Python et pip installés sur votre machine, vous pouvez exécuter cette application directement.

1.  **Vérifiez vos versions (optionnel)**
    ```bash
    $ python3 --version
    $ pip3 --version
    ```

2.  **Installez Flask**
    ```bash
    $ pip3 install flask
    ```

3.  **Lancez l'application**
    ```bash
    $ python3 app.py
     * Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)
    ```

---

# Part 5 : Construire une image Docker

L'un des avantages des conteneurs est que vous pouvez intégrer toutes les dépendances (comme Python) dans votre image, sans avoir besoin de les installer sur votre hôte.

## 1. Créer un Dockerfile

Un **`Dockerfile`** est un document texte qui contient toutes les commandes pour assembler une image.

1.  Créez le fichier :
    ```bash
    $ touch Dockerfile
    ```
2.  Ajoutez le contenu suivant dans le `Dockerfile` (par exemple, avec `vi Dockerfile` ou un autre éditeur) :
    ```dockerfile
    # Image de base
    FROM python:3.6.1-alpine

    # Installation des dépendances
    RUN pip install --upgrade pip
    RUN pip install flask

    # Copie du code de l'application
    COPY app.py /app.py

    # Commande à exécuter au démarrage du conteneur
    CMD ["python", "app.py"]
    ```

### Comprendre les instructions du Dockerfile

- **`FROM python:3.6.1-alpine`**: Définit l'image de base. `alpine` est une distribution Linux légère, ce qui rend l'image plus petite et plus sécurisée. Il est recommandé d'utiliser des **images officielles** et des **tags spécifiques** (comme `3.6.1-alpine` plutôt que `latest`) pour la stabilité et la sécurité.

- **`RUN pip install ...`**: Exécute des commandes pendant la construction de l'image. Chaque `RUN` crée une nouvelle couche dans l'image. Ici, on installe Flask.

- **`COPY app.py /app.py`**: Copie les fichiers de votre répertoire local dans l'image. Il est recommandé de placer les instructions `COPY` le plus bas possible dans le Dockerfile. Les couches changent fréquemment (comme le code source) et en les plaçant à la fin, on tire parti du **cache de couches** de Docker, ce qui accélère les builds suivants.

- **`CMD ["python", "app.py"]`**: Définit la commande par défaut qui sera exécutée au démarrage du conteneur. Il ne peut y avoir qu'une seule instruction `CMD` par Dockerfile.

## 2. Construire l'image

Une fois le `Dockerfile` créé, construisez l'image.

```bash
# Le '.' à la fin indique que le contexte de build est le répertoire actuel
$ docker image build -t python-hello-world .
```
- **`-t python-hello-world`**: Nomme (`-t` pour tag) l'image `python-hello-world`.

Docker exécutera chaque étape du `Dockerfile` et affichera la progression.

## 3. Vérifier l'image

Listez vos images Docker pour vérifier que la nouvelle image a bien été créée.
```bash
$ docker image ls
```
Vous devriez voir `python-hello-world` et son image de base `python:3.6.1-alpine` dans la liste.

---

# Part 6 : Exécuter votre image personnalisée

## 1. Lancer le conteneur

Maintenant que l'image est construite, vous pouvez l'exécuter.

```bash
$ docker run -p 5001:5000 -d python-hello-world
```
- **`-p 5001:5000`**: Mappe (`-p` pour publish) le port `5001` de votre machine hôte au port `5000` exposé par le conteneur. Si le port 5001 est déjà pris, vous pouvez utiliser un autre port (ex: 5002).
- **`-d`**: Exécute le conteneur en mode détaché (en arrière-plan).

## 2. Vérifier l'application

Ouvrez votre navigateur et allez à l'adresse **http://localhost:5001**. Vous devriez voir le message "hello world!".

## 3. Consulter les journaux (logs)

Pour voir la sortie de votre application à l'intérieur du conteneur, utilisez la commande `docker container logs`.

1.  Trouvez l'ID de votre conteneur :
    ```bash
    $ docker container ls
    ```
2.  Affichez les journaux :
    ```bash
    $ docker container logs <ID_DU_CONTENEUR>
    # Sortie attendue
    * Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)
    172.17.0.1 - - [28/Jun/2017 19:35:33] "GET / HTTP/1.1" 200 -
    ```

## 4. Prochaines étapes : CI/CD

Le `Dockerfile` permet des constructions reproductibles. Un flux de travail courant en CI/CD (Intégration Continue/Déploiement Continu) consiste à :
1.  Construire l'image Docker (`docker image build`) automatiquement.
2.  Pousser l'image vers un registre central (comme le Docker Hub).
3.  Déployer l'image depuis le registre vers les différents environnements (test, production, etc.).

---

# Part 7 : Partager votre image sur Docker Hub

Pour partager votre image, vous utiliserez **Docker Hub**, un service de registre qui permet de stocker des images publiquement (gratuit) ou en privé (payant).

## 1. Se connecter à Docker Hub

- **Créez un compte** : Si ce n'est pas déjà fait, créez un compte gratuit sur [Docker Hub](https://hub.docker.com).
- **Connectez-vous** via le terminal :
  ```bash
  $ docker login
  ```
  Suivez les instructions pour entrer votre nom d'utilisateur et mot de passe.

## 2. Taguer l'image

Pour pousser une image sur Docker Hub, vous devez la taguer en respectant la convention de nommage : `<nom_utilisateur_dockerhub>/<nom_image>`.

```bash
$ docker tag python-hello-world <nom_utilisateur_dockerhub>/python-hello-world
```
Remplacez `<nom_utilisateur_dockerhub>` par votre propre nom d'utilisateur.

## 3. Pousser (Push) l'image

Utilisez la commande `docker push` pour envoyer votre image taguée vers le registre Docker Hub.

```bash
$ docker push <nom_utilisateur_dockerhub>/python-hello-world
```

## 4. Vérifier sur Docker Hub

Connectez-vous à votre compte Docker Hub dans votre navigateur. Vous devriez voir le nouveau dépôt `python-hello-world` dans votre liste.

Maintenant, n'importe qui peut récupérer et utiliser votre image avec la commande `docker pull <nom_utilisateur_dockerhub>/python-hello-world`. Cela simplifie le déploiement car l'image contient toutes les dépendances nécessaires, éliminant les problèmes de "dérive d'environnement" entre différentes machines.

---

# Part 8 : Déployer un changement

## 1. Mettre à jour le code

Modifiez le fichier `app.py` pour changer le message retourné.

```python
from flask import Flask

app = Flask(__name__)

@app.route("/")
def hello():
    return "Hello Beautiful World!"

if __name__ == "__main__":
    app.run(host='0.0.0.0')
```

## 2. Reconstruire l'image

Reconstruisez l'image en utilisant le tag complet (incluant votre nom d'utilisateur Docker Hub).

```bash
$ docker image build -t <nom_utilisateur_dockerhub>/python-hello-world .
```
Vous remarquerez que Docker utilise le cache pour les premières couches qui n'ont pas changé :
```
Étape 1/4 : FROM python:3.6.1-alpine
 ---&gt; c86415c03c37
Étape 2/4 : RUN pip install flask
 ---&gt; Utilisation du cache
 ---&gt; ce41f2517c16
...
Étape 4/4 : COPY app.py /app.py
 ---&gt; 3e08b2eeace1
```
Seule la couche `COPY` et les suivantes sont reconstruites, car c'est là que le changement a eu lieu. C'est pourquoi l'ordre des instructions dans un `Dockerfile` est crucial pour optimiser les temps de construction. Les couches qui changent le plus souvent (comme le code source) doivent être placées le plus bas possible.

## 3. Pousser la mise à jour

Poussez la nouvelle version de l'image vers le Docker Hub.

```bash
$ docker push <nom_utilisateur_dockerhub>/python-hello-world
```
Là encore, le cache est utilisé. Docker et Docker Hub détectent que la plupart des couches existent déjà et n'envoient que la nouvelle couche modifiée, ce qui rend le processus très rapide.

---

# Part 9 : Comprendre les couches d'images (Union File System)

Le **système de fichiers union** (UnionFS) est une propriété de conception fondamentale de Docker.

Chaque instruction dans un `Dockerfile` crée une **couche**. Chaque couche ne contient que les changements (le delta) par rapport à la couche précédente. Docker utilise le système de fichiers union pour superposer ces couches de manière transparente en une seule vue unifiée.

- **Couches en lecture seule (Read-Only)** : Toutes les couches d'une image sont en lecture seule.
- **Couche de conteneur** : Lorsque vous lancez un conteneur, Docker ajoute une couche supérieure inscriptible.
- **Copy-on-Write** : Si vous modifiez un fichier qui existe dans une couche inférieure (lecture seule), Docker copie ce fichier dans la couche supérieure inscriptible et y applique la modification. Le fichier original dans la couche inférieure reste inchangé. Ce mécanisme est appelé "copy-on-write".

### Partage des couches

Comme les couches d'image sont en lecture seule, elles peuvent être partagées.
- **Entre images** : Si plusieurs images partagent les mêmes couches de base (ex: `FROM python:3.6.1-alpine`), ces couches ne sont stockées qu'une seule fois sur l'hôte.
- **Entre conteneurs** : Plusieurs conteneurs démarrés à partir de la même image partagent les mêmes couches en lecture seule, et chacun obtient sa propre couche inscriptible. Cela rend le démarrage des conteneurs extrêmement rapide et efficace en termes d'espace disque.

Ce mécanisme de superposition est aussi ce qui rend le cache de `build` et de `push` si efficace.

### Inspecter les couches

Vous pouvez examiner l'historique des couches d'une image avec la commande `docker image history`.

```bash
$ docker image history python-hello-world
```
La sortie montre chaque couche, la commande qui l'a créée, et sa taille. Les lignes supérieures correspondent à votre `Dockerfile`, et les lignes inférieures proviennent de l'image de base `python:3.6.1-alpine`.

```
IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
f1b2781b3111        About a minute ago   /bin/sh -c #(nop) COPY file:0114358808a1bb...   159B                
0ab91286958b        About a minute ago   /bin/sh -c #(nop) CMD ["python" "app.py"]       0B                  
ce41f2517c16        About a minute ago   /bin/sh -c pip install flask                     10.6MB              
c86415c03c37        8 days ago          /bin/sh -c #(nop) CMD ["python3"]               0B                  
<missing>           8 days ago          /bin/sh -c set -ex;   apk add --no-cache -...   5.73MB              
...
```
Les balises `<missing>` indiquent des couches qui n'ont pas reçu d'ID lisible par le système Docker local, mais ce sont des couches normales héritées de l'image de base.

---

# Part 10: Créer votre premier swarm

Dans cette section, vous allez créer votre premier swarm en utilisant Play-with-Docker.

Naviguez vers Play-with-Docker. Vous allez créer un swarm avec trois nœuds.

Cliquez sur Ajouter une nouvelle instance à gauche trois fois pour créer trois nœuds. Initialisez le swarm sur le nœud 1 :
```
$ docker swarm init --advertise-addr eth0
```
Swarm initialisé : le nœud actuel (vq7xx5j4dpe04rgwwm5ur63ce) est maintenant un gestionnaire.

Pour ajouter un worker à ce swarm, exécutez la commande suivante :
```
docker swarm join \
--token SWMTKN-1-50qba7hmo5exuapkmrj6jki8knfvinceo68xjmh322y7c8f0pj-87mjqjho30uue43oqbhhthjui \
10.0.120.3:2377
```
Pour ajouter un gestionnaire à ce swarm, exécutez 'docker swarm join-token manager' et suivez les instructions. Vous pouvez considérer Docker Swarm comme un mode spécial activé par la commande : `docker swarm init`. L'option `--advertise-addr` spécifie l'adresse que les autres nœuds utiliseront pour rejoindre le swarm.

Cette commande `docker swarm init` génère un jeton de jointure. Le jeton garantit qu'aucun nœud malveillant ne rejoint le swarm. Vous devez utiliser ce jeton pour joindre les autres nœuds au swarm. Pour plus de commodité, la sortie comprend la commande complète `docker swarm join`, que vous pouvez simplement copier/coller dans les autres nœuds.

Sur les nœuds 2 et 3, copiez et exécutez la commande `docker swarm join` qui a été affichée dans votre console par la dernière commande. Vous avez maintenant un swarm de trois nœuds !

Retournez sur le nœud 1, exécutez `docker node ls` pour vérifier votre cluster à trois nœuds :
```
$ docker node ls
ID                          HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS
7x9s8baa79l29zdsx95i1tfjp   node3               Ready               Active              
x223z25t7y7o4np3uq45d49br   node2               Ready               Active              
zdqbsoxa6x1bubg3jyjdmrnrn *  node1               Ready               Active              Leader
```
Cette commande affiche les trois nœuds de votre swarm. L’astérisque (*) à côté de l’ID du nœud représente le nœud qui a géré cette commande spécifique (`docker node ls` dans ce cas).

Votre nœud se compose d'un nœud gestionnaire et de deux nœuds workers. Les gestionnaires gèrent les commandes et l'état du swarm. Les workers ne peuvent pas gérer les commandes et sont simplement utilisés pour exécuter des conteneurs à grande échelle. Par défaut, les gestionnaires sont également utilisés pour exécuter des conteneurs.

Toutes les commandes `docker service` pour le reste de ce laboratoire doivent être exécutées sur le nœud gestionnaire (Nœud 1).

Remarque : Bien que vous contrôliez le swarm directement depuis le nœud sur lequel il s'exécute, vous pouvez contrôler un swarm Docker à distance en vous connectant au moteur Docker du gestionnaire en utilisant l'API distante ou en activant un hôte distant depuis votre installation Docker locale (en utilisant les variables d'environnement `$DOCKER_HOST` et `$DOCKER_CERT_PATH`). Cela sera utile lorsque vous souhaiterez contrôler à distance des applications de production, au lieu d'utiliser SSH pour contrôler directement des serveurs de production.

---

# Part 11 : Déployer et gérer des services Swarm

Maintenant que vous avez votre cluster Swarm à trois nœuds initialisé, vous allez déployer des conteneurs. Pour exécuter des conteneurs sur un Docker Swarm, vous devez créer un **service**. Un service est une abstraction qui représente plusieurs conteneurs de la même image déployés sur un cluster distribué.

## 1. Créer un service

Sur **Node1** (nœud manager), déployez un service en utilisant NGINX :

```bash
$ docker service create --detach=true --name nginx1 --publish 80:80 --mount source=/etc/hostname,target=/usr/share/nginx/html/index.html,type=bind,ro nginx:1.12
```

- **Déclaratif** : Cette commande est déclarative. Docker Swarm essaiera de maintenir l'état déclaré (1 conteneur nginx) à moins qu'il ne soit explicitement modifié.
- **`--mount`** : Ici, nous montons le fichier `/etc/hostname` de l'hôte dans le fichier `index.html` de NGINX. Cela permettra d'afficher le nom du nœud qui répond à la requête, utile pour visualiser l'équilibrage de charge.
- **`--publish` (Routing Mesh)** : Le port 80 est exposé sur **chaque nœud** du swarm. Le maillage de routage (routing mesh) orientera une demande provenant du port 80 de n'importe quel nœud vers l'un des nœuds exécutant le conteneur.

### Vérifier le service

Inspectez le service créé :
```bash
$ docker service ls
```

Pour examiner les tâches (conteneurs) du service :
```bash
$ docker service ps nginx1
```
Vous verrez sur quel nœud le conteneur a été planifié.

### Tester le service

Grâce au maillage de routage, vous pouvez envoyer une requête à n'importe quel nœud (localhost, IP node1, IP node2, etc.) sur le port 80.

```bash
$ curl localhost:80
```
Le retour affichera le nom d'hôte (ex: `node1`) où le conteneur s'exécute réellement.

## 2. Écheller votre service (Scaling)

En production, pour gérer plus de trafic, vous pouvez écheller le service.

Mettez à jour le service pour avoir 5 réplicas :
```bash
$ docker service update --replicas=5 --detach=true nginx1
```

Docker Swarm reconnaît que l'état désiré (5 réplicas) ne correspond pas à l'état actuel (1 réplica) et planifie 4 nouveaux conteneurs. La stratégie par défaut place les conteneurs sur les nœuds les moins occupés.

Vérifiez la répartition :
```bash
$ docker service ps nginx1
```

### Tester l'équilibrage de charge

Envoyez plusieurs requêtes :
```bash
$ curl localhost:80
$ curl localhost:80
$ curl localhost:80
```
Vous devriez voir les noms des différents nœuds (`node1`, `node2`, `node3`) apparaître en alternance. Le Routing Mesh agit comme un équilibreur de charge.

**Voir les logs agrégés :**
Vous pouvez voir les logs de tous les conteneurs du service en une seule commande :
```bash
$ docker service logs nginx1
```

## 3. Appliquer des mises à jour progressives (Rolling Updates)

Nous allons mettre à jour la version de NGINX de `1.12` à `1.13` sans arrêter le service.

```bash
$ docker service update --image nginx:1.13 --detach=true nginx1
```

Cela déclenche une mise à jour progressive. Swarm met à jour les conteneurs un par un (ou par lots selon la configuration).

Vous pouvez suivre l'avancement avec :
```bash
$ docker service ps nginx1
```
Vous verrez les anciennes versions s'arrêter et les nouvelles démarrer.

## 4. Réconciliation et tolérance aux pannes

Docker Swarm utilise un modèle "inspecter puis adapter". Si un nœud tombe en panne, Swarm détecte la perte des conteneurs et les reprogramme sur des nœuds sains pour respecter l'état désiré (nombre de réplicas).

### Simulation d'une panne

1.  Créez un nouveau service pour ce test (pour avoir une sortie propre) :
    ```bash
    $ docker service create --detach=true --name nginx2 --replicas=5 --publish 81:80 --mount source=/etc/hostname,target=/usr/share/nginx/html/index.html,type=bind,ro nginx:1.12
    ```

2.  Sur **Node1**, surveillez le service :
    ```bash
    $ watch -n 1 docker service ps nginx2
    ```
    *(Si `watch` n'est pas disponible, répétez simplement la commande `docker service ps nginx2`)*.

3.  Sur **Node3**, quittez le swarm pour simuler une panne :
    ```bash
    $ docker swarm leave
    ```

4.  Observez sur **Node1** :
    Vous verrez que les tâches qui tournaient sur `node3` passent à l'état `Shutdown` ou `Failed`, et que Swarm redémarre immédiatement de nouvelles instances sur `node1` et `node2` pour revenir à 5 réplicas actifs.

### Note sur les nœuds Managers et le Quorum

Dans ce laboratoire, nous avons 1 Manager et 2 Workers. Ce n'est pas une configuration haute disponibilité (HA). Si le Manager tombe, le cluster ne peut plus être administré.

Pour la production :
- Il faut un nombre impair de managers pour le consensus (algorithme Raft).
- **3 Managers** : Tolèrent 1 panne.
- **5 Managers** : Tolèrent 2 pannes.
- **7 Managers** : Tolèrent 3 pannes.

Il est recommandé de ne pas dépasser 7 managers pour éviter les latences de consensus, mais vous pouvez avoir des milliers de nœuds workers.