# PROJET INFRA SI


## Prérequis
                        
- Debian 10 
- une carte réseau NAT
- une carte réseau privé hôte 

## Installation de Seafile 

Pour installer un serveur de partage de fichier, commencez par installer le dernier package server : 
```
wget https://s3.eu-central-1.amazonaws.com/download.seadrive.org/seafile-server_8.0.4_x86-64.tar.gz
```

Pour l’organisation, nous allons créer un dossier qui sera ici nommé “SIprojet” : 
```
mkdir SIprojet
mv seafile-server_* SIprojet
cd SIprojet
tar -xzf seafile-server_*
mkdir installed
mv seafile-server_* installed
```

Pour installer quelques packages requis, nous allons éxécuter ces commandes en admin :

```
apt-get update
apt-get install python2.7 libpython2.7 python-setuptools python-ldap python-urllib3 sqlite3 python-requests
```

Puis en tant qu'utilisateur, pour configurer le serveur, il faudra lancer ce script : 
```
cd seafile-server-*
./setup-seafile.sh
```

Il vous faudra répondre à quelques questions.
Vous devrez inscrire :
- le nom du serveur
- l'IP de cette machine
- le port qu'utilisera seafile (défault ou à changer ???)

Ensuite, il faudra démarrer le serveur : 
```
ulimit -n 30000
./seafile.sh start
./seahub.sh start
```
La première fois, il vous sera demandé de créer un compte admin pour le serveur seafile.
Il faudra entrer : 
- Une adresse e-mail
- Un mot de passe
- La confirmation de ce dernier

Le lancement du serveur échouera et vous demandera de le redémarrer.

En admin, installez donc les paquets nécessaires :
```
apt install python3-pip
pip3 install django-recaptcha
apt remove apache2
apt install nginx
rm /etc/nginx/sites-enabled/default
ln -s /etc/nginx/sites-available/seafile.conf /etc/nginx/sites-enabled/seafile.conf
pip3 install --ignore-installed --timeout=3600 Pillow captcha jinja2 sqlalchemy psd-tools django-pylibmc python3-ldap
pip3 install --ignore-installed django-simple-captcha
```

Revenez en user et reconfigurez ce fichier, vous devez rajouter le port du serveur après votre IP : 
```
nano conf/ccnet.conf
```
```
SERVICE_URL = http://votreIP:8000
```

Puis reconfigurez aussi ce fichier, il faut changer son adresse IP par la vôtre : 
```
nano conf/gunicorn.conf.py
```
```
#default localhost:8000
bind = "votreIP:8000"
```

Il reste d'autres fichiers à créer et à configurer :

Dans celui-ci : 
```
nano conf/gunicorn.conf
```
Ecrivez : 
```
#default localhost:8000"
bind = "0.0.0.0:8000
```
Et dans celui-là : 
```
nano conf/seahub_settings.py
```
Rajoutez la ligne : 
```
FILE_SERVER_ROOT = 'http://seafile.example.com/seafhttp'
```

Rdémarrez le serveur :
```
./seafile.sh restart
./seahub.sh restart
```

Puis entrez dans votre navigateur : 
```
http://votreIP:8000
```

Vous avez ainsi accès un votre serveur de partage de fichier !


## Installation de NetData

Sur votre terminal, connectez-vous en admin : 
```
su -
```
Puis lancez cette unique commande : 
```
bash <(curl -Ss https://my-netdata.io/kickstart.sh)
```
Il sera nécessaire de valider de temps en temps en appuyant sur la touche "entrée".
Une fois installé, Netdata est automatiquement démarré et activé sur systemd.

Vous pouvez vérifier son état grâce à la commande : 
```
systemctl status netdata
```

Vous pouvez désormais accéder à votre interface web grâce à "votreIP":19999

**Créer des alertes sur discord**

Sur votre serveur discord allez dans les paramètres de votre serveur, puis dans WEBHOOKS, et faites Nouveau webhook. Choississez le nom et le salon dans lequel il enverra les alertes.

Une fois terminé cliquez sur Copier l'URL du webhook.

Sur votre terminal écrivez cette commande pour modifier le fichier de configuration des alertes :
```
/etc/netdata/edit-config health alarm notify.conf
```

Une fois dans la zone de discord sur le fichier config mettez l'URL que vous avez copié tout à l'heure dans DISCORD_WEBHOOK_URL=""
ensuite sauvergarder en faisant CTRL+X.

Une fois la configuration terminée redémarrez NetData en faisant un :
```
systemctl stop netdata
```
```
systemctl start netdata
```

Pour ensuite créer des alertes vous devrez vous rendre dans le dossier health.d pour cela faites :
```
cd etc/
cd netdata
```
Nous allons créer une alarme pour le CPU donc une fois dans ce dossier faites :
```
./edit-config health.d/cpu.conf
```

Les lignes warn et crit peuvent être modifiées pour changer le seuil minimum et maximum pour activer les alertes.

Une fois terminé sauvegardez.

Vérifiez que le dossier est bien créé dans :
```
cd health.d
```

Ensuite redémarrez votre NetData.
