# Demo : déployer une application python

L'objectif de ce tutoriel est de proposer un processus de déploiement d'une application, de la façon la plus automatisée possible, du début (le code source) à la fin (application exposée sur internet).

1. Les sources

Il convient de versionner le code sur un outil de versionning. Actuellement, le standard est `git`. On considère dans la suite que le code est disponible sur un dépôt, chez l'hébergeur de votre choix (les plus populaires étant `gitlab` et `github`).

A noter : il existe un certain nombre de bonnes pratiques sur le versionning. Essayons de les résumer brièvement :

- Le versionning n'est efficace que pour du texte. Le code source a donc toute sa place sur git mais ce n'est pas le cas des données (hors jeu de test minimal), des photos / vidéos / fichiers binaires (privilégier le stockage `S3` pour ces fichiers) ou d'éventuels documents non texte (pdf, odt, doc ...). Pour la documentation, elle a sa place sur le dépôt à condition d'utiliser un format en texte brut comme le `markdown`.
- Il ne faut versionner que les fichiers nécessaires à la reproductibilité. Les fichiers produits (exemple : output, fichiers compilés, cache ...) n'ont pas leur place sur le repo git. L'utilisation du fichier `.gitignore` permet de dire à `git` d'ignorer les fichiers qui n'ont pas vocation à être versionnés.

Le dépôt `git` joue le rôle de **source de vérité** sur l'état du code et de point de départ pour le processus de packaging / déploiement que nous allons voir ici.

2. Déploiement en local

Avant de s'intéresser au déploiement de notre application sur un serveur, il convient de vérifier qu'elle s'éxecute bien localement sur le poste d'un développeur.  
Ce projet utilisant [fastapi](https://fastapi.tiangolo.com/), il peut être lancé de la manière suivante :

```
pip install requirements.txt # installation des dépendances nécessaires
uvicorn main:app --reload # le --reload convient pour une utilisation en développement, pas utile en production
```

L'application se lance et écoute bien sur le port `8000`. On peut donc ouvrir un navigateur et accéder à l'application sur `http://localhost:8000`

3. Tests

Bien qu'ils ne soient pas strictement nécessaires au déploiement, les tests jouent un rôle crucial dans l'automatisation et la fiabilisation du processus de déploiement.

4. Packaging

Le packaging consiste à produire un livrable autonome à partir du code source. Ce livrable doit permettre d'exécuter l'application sans avoir besoin de la recompiler, de s'intéresser au code source et avec le moins de prérequis possibles.  
En fonction du langage, le format du package peut changer et les étapes pour packager peuvent différer (résolution des dépendances, compilation, zippage, ajout de métadonnées, signature ...).  
Python étant un langage interprété (et non compilé), l'étape de packaging n'est pas strictement nécessaire. Comme on l'a vu précédemment, il suffit d'avoir python (et pip) installé pour exécuter notre application à partir de son code source.

Même s'il peut paraitre léger, le prérequis d'avoir python préinstallé sur le serveur qui va déployer l'application se révèle souvent être un frein majeur à la portabilité et à la maintenabilité de l'application. En effet, on se retrouve obligé de préparer l'environnement cible spécifiquement pour notre application. Par exemple, les montées de version de python ne sont alors pas uniquement de la responsabilité des développeurs de l'application mais nécessitent une coordination avec chaque environnement de déploiement.  
Pour cette raison (et bien d'autres qui dépassent le cadre de ce tutoriel), un nouveau type de livrable "universel" s'est imposé ces dernières années : l'image `docker`.

5. Packaging docker

Comme on a vu précédemment, on souhaite réaliser et publier une image `docker` de notre application afin d'avoir un livrable autonome, ne nécessitant aucun prérequis (autre que docker - ou tout autre moteur de conteneurisation - lui même) sur les environnements de déploiement. Cette image contiendra à la fois le code de notre application mais aussi la version spécifique de python qui convient.

La définition d'une image docker se fait au travers d'un fichier `Dockerfile`.  
Ce fichier [cf Dockerfile](Dockerfile) contient le listing de tout ce qui est nécessaire pour faire tourner notre application (python, dépendances, code, ligne de commande d'exécution) afin de produire un livrable totalement autonome.

Si vous avez Docker installé sur votre machine, vous pouvez construire localement l'image :

```
docker build -t monapplication .
```

Puis l'exécuter localement :

```
docker run -p 8000:8000 monapplication
```

Votre application est alors accessible sur `http://localhost:8000`

Pas de panique si vous n'avez pas docker ! Nous allons tout faire via l'intégration continue dans un chapitre ultérieur.

6. Publication de l'image Docker sur internet

Notre livrable est maintenant prêt ! On peut maintenant le publier au monde entier (ou en interne dans une organisation si le projet n'est pas public). Pour cela, on va publier (push) notre image sur un registre docker.
Le plus connu est [Dockerhub](https://hub.docker.com/) mais il en existe d'autres. Pour Dockerhub, il est nécessaire de créer un compte (gratuit) afin de publier des images. Pour du testing ou si la création de compte n'est pas possible, on peut utiliser le registre public [ttl.sh](https://ttl.sh/) qui permet la publication sans inscription mais avec une durée de conservation maximale de 24h.

Si vous avez Docker installé localement, vous pouvez publier avec les commandes suivantes :

Pour dockerhub

```
docker build -t monpseudodocker/monapp .
docker login
docker push monpseudodocker/monapp
```

Pour ttl.sh

```
docker build -t ttl.sh/unidentifiantunique:24h
docker push ttl.sh/unidentifiantunique:24h
```

L'application est maintenant disponible publiquement !  
Toute personne qui a docker (ou tout autre moteur de conteneurisation) peut maintenant l'exécuter localement avec la commande

```
docker run -p 8000:8000 monpseudodocker/monapp # pour dockerhub
docker run -p 8000:8000 ttl.sh/unidentifiantunique:24h # pour ttl.sh
```

7. Déploiement de l'application  

Maintenant que le livrable de notre application est disponible publiquement, on peut la déployer sur n'importe quel environnement Docker (ou tout autre moteur de conteneurisation).  
Le standard du déploiement qui s'est imposé ces dernières années est `Kubernetes` qui est un orchestrateur de conteneurs. C'est la version macro de Docker. Il orchestre les conteneurs à l'échelle d'un cluster de machines (plusieurs machines avec Docker - ou tout autre moteur de conteneurisation - regroupées au sein d'un même groupe).  
Cela permet un bien meilleur passage à l'échelle, une facilité de gestion courante et un tas d'autres avantages tout en gardant le principe de la conteneurisation.  

L'installation d'un cluster Kubernetes va bien au dela de ce tutoriel mais, bonne nouvelle, un cluster vous est mis à disposition au travers du datalab SSPCloud.  

Rendez vous sur le SSPCloud pour déployer un service `VSCode` en choisissant `admin` dans l'onglet `Kubernetes`. [Lien direct](https://datalab.sspcloud.fr/launcher/inseefrlab-helm-charts-datascience/vscode?autoLaunch=false&kubernetes.role=%C2%ABadmin%C2%BB). Ce choix `admin` donne à votre service `VSCode` les permissions nécessaires au déploiement d'une application dans le cluster.  

Une fois le `VSCode` lancé, ouvrez le et ouvrez y un terminal.  
Dans ce terminal, vous pouvez récupérer le code de votre dépôt (nous utiliserons ici le dépôt de ce tutoriel) :  
```
git clone https://github.com/olevitt/demo-deploiement.git
```  

