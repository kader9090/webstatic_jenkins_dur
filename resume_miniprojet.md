Utilisation jenkins pour CICD
Installation : 
* docker (dirane)
* sinon installation en dur (documentation officiel)

Infrastructure:
1) une machine : pour server jenkins ( qui permet de builder et tester en utilisant un agent any ou agent docker, et on deploit sur cloud (heroku)
2) deux machines : 1e pour server jenkins ( qui permet de builder et tester en utilisant un agent any ou agent docker, et on deploit sur la deuxieme machine prod ou bien
deux machines : 1e pour server jenkins qui va envoyer les taches en ssh  et les executer sur la deuxieme machine : builder et tester en utilisant un agent any ou agent docker, et on deploit sur la deuxieme machine prod


Dans ce cas de ùini_projet, on utilisera la troisième méthode. cad deux machines, une pour jenkins et une deuxieme pour production.

on clone le prjet de Dirane en local
git clone

on crée un repo sur mon compte github et on push le projet cloné

git remote set-url origin https://github.com/kader9090/webstatic_jenkins_dur.git
git branch -M main
git push -u origin main
----------------
# machine1: kader-ec2-prod: ubuntu 20.04 lts, t2.micro, SSD 20 Go, securité groupe : 22/8080:interface Jenkins/80 :siteweb

script:
bash
```
#!/bin/bash
curl -fsSL https://get.docker.com -o get-docker.sh
sh get-docker.sh
sudo usermod -aG docker ubuntu
```
je me connecte en ssh sur la machine prod pour vérifier si docker est bien installé:
docker --version => Docker version 20.10.12, build e91ed57 => docker est bien installé

pour vérifier que curl est bien présent sur la machine (ça sera utilisé pour l'acceptation dans le jenkinsfile)

curl --version => curl 7.68.0 (x86_64-pc-linux-gnu) libcurl/7.68.0 OpenSSL/1.1.1f zlib/1.2.11 brotli/1.0.7 libidn2/2.2.0 libpsl/0.21.0 (+libidn2/2.2.0) libssh/0.9.3/openssl/zlib nghttp2/1.40.0 librtmp/2.3
Release-Date: 2020-01-08

 

# Installation de Jenkins:
# machine2:kader-ec2-jenkins : ubuntu 20.04 lts, t2.large, SSD 20 Go, securité groupe : 22/8080:interface Jenkins
procédure :

 Déployez un serveur Jenkins (from scratch) et non à l’aide de docker sur une instance AWS ec2 
(https://www.digitalocean.com/community/tutorials how-to-install-jenkins-on-ubuntu-20-04-fr )

sudo apt install default-jre
java --version => 
openjdk version "11.0.13" 2021-10-19
OpenJDK Runtime Environment (build 11.0.13+8-Ubuntu-0ubuntu1.20.04)
OpenJDK 64-Bit Server VM (build 11.0.13+8-Ubuntu-0ubuntu1.20.04, mixed mode)


sudo apt install jenkins -y
sudo systemctl start jenkins
sudo systemctl status jenkins

![Capture203](./Capture1.JPG)


sudo ufw allow 8080  =>

Rules updated
Rules updated (v6)

sudo ufw status

Status: inactive

Installation Docker

curl -fsSL https://get.docker.com -o get-docker.sh
sh get-docker.sh
sudo usermod -aG docker ubuntu
sudo usermod -aG docker jenkins

# Configuration de Jenkins

http://3.80.141.120:8080/



![Capture203](./Capture2.JPG)


sudo cat /var/lib/jenkins/secrets/initialAdminPassword


![Capture203](./Capture3.JPG)

on installe les plugins suggérés et on crée un utilisateur

créer un nouveau job => Pipeline 

je configure le projet pipeline, 
GitHub project: https://github.com/kader9090/webstatic_jenkins_dur.git/
Build Triggers: GitHub hook trigger for GITScm polling
Pipeline: regardes capture

![Capture203](./Capture4.JPG)
![Capture203](./Capture5.JPG)

# ajouter des credentials: 

token github:
 * Global
    * Nom : kader9090
    * MDP : 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
    * ID  : github_password
    * descritpion  : github_password


* Docker Hub :
    * Global
    * Nom : abdelkader90
    * MDP : cefbd139-c194-4eed-95ef-0f8fe04d640c
    * ID  : dockerhub_password

* clé SSH pour Production :
    * SSH Username with private key
    * ID : kader_ec2_private_key
    * Username : ubuntu
    * Private key : celle qu'on utilise pour se connecter sur nos machines AWS



# ajouter des plugins:

*Github Intégration
Administrer Jenkins → Plugin → Installer GitHub Integration

configuration du webhook sur le github
[ GitHub ] : Projet → Settings → add webhook

payload url:
@ip de server jenkins:8080/github-webhook/

![Capture203](./Capture6.JPG)

on teste le lien webhook comme indiqué sur cette capture:

![Capture203](./Capture7.JPG)



*embedaable-build-status
[ Jenkins ] → Administrer Jenkins → Plugin → Embeddable-build-status

ajouter le github build status dans le README.md
[ Jenkins ] → cicd-jenkins_dur-webstatic → Embeddable Build Status → Links / Markdown / unprotected=> je copie le lien unprotected et je colle sur le README.md sur vscode.

![Capture203](./Capture9.JPG)

*slack notification Plugin
[ Jenkins ] → Administrer Jenkins → Plugin → Slack Notification Plugin

sur slack:
    * Création d'un nouveau canal : jenkinsdur_webstat
    * Propriétés ==> Intégration ==> Application ==> Ajouter une application
        * choisir Jenkins ==> configuration
        * Ajouter à Slack
        * Publier dans le canal : jenkinsdur_webstat
        * Ajouter
        * Récupérer les données :
            * canal : #jenkinsdur_webstat
Sous-domaine de l’équipe: ajc-devops-siege
Identifiant d’authentification du jeton d’intégration: Hm7tGw2JjRajIxPCmQCOHgXV

* Enregistrer les paramètres


configuration de slack sur Jenkins 
sur jenkins: configurer le systeme=> slack => créer credential =>ajouter credential de type secret texte

workspace: ajc-devops-siege (Sous-domaine de l’équipe)
le canal :jenkinsdur-webstatic

![Capture203](./Capture8.JPG)

le test de connexion est reussi

# Dockerfile:

```
FROM nginx
COPY ./ /usr/share/nginx/html
```

# Jenkinsfile

Maintenant que la configuration de l'environnement est terminée, on écrit le Jenkinsfile:

```


```
