# installation  jitsi version 2.9 

+ `Introduction :`

Packages JITSI à installer  (uniquement ceux -la)
jitsi-videobridge
jicofo
jitsi-meet-prosody
jitsi-meet
 
Installation : (ordre à respecter)

+ `OS :`

WM os linux VM locale Ubuntu 16.10

+ `Java :`

openjdk version "1.8.0_102"
sudo apt-get install openjdk-8-jdk
sudo apt-get install openjdk-8-jre-headless

+ `php :`

sudo apt install php7.0-cli
sudo apt-get -y install php7.0-fpm
modifier le fichier /etc/php/7.0/fpm/php.ini
cgi.fix_pathinfo=0   (enlever le commentaire « ; » et le mettre à 0)
puis redémarrer:
sudo systemctl restart php7.0-fpm
curl: sudo apt-get install php-curl

+ `NGINX :`

1) Installer  nginx
sudo apt-get install nginx
Lanceur dans /etc/init.d/nginx.
Binaire dans /usr/sbin/nginx.
Pid dans /run/nginx.pid.
Defaults dans /etc/default/nginx.
Configuration dans /etc/nginx/.
Logs dans /var/log/nginx/.

2 ) Génération d'un certificat SSL (avec  l’outil Cerbot)
 
cd /usr/local/sbin
sudo wget https://dl.eff.org/certbot-auto
sudo chmod a+x /usr/local/sbin/certbot-auto
sudo ./certbot-auto certonly -a webroot --webroot-path=/usr/share/jitsi/jitsi-meet/ -d vtr-preprod.spoc.pro
puis ajouter dans le fichier /etc/nginx/sites-available/vtr-preprod.spoc.pro.conf :
     ssl_certificate /etc/letsencrypt/live/vtr-preprod.spoc.pro/cert.pem;
     ssl_certificate_key /etc/letsencrypt/live/vtr-preprod.spoc.pro/privkey.pem;

enfin ajouter dans le cron :
sudo crontab -e
30 2 * * 1 /usr/local/sbin/certbot-auto renew >> /var/log/le-renew.log
35 2 * * 1 /etc/init.d/nginx reload
 
 
 
3) Configurer nginx
/etc/nginx/sites-available : vtr-preprod.spoc.pro.conf
 
4) Activer la configuration
sudo ln -s /etc/nginx/sites-available/vtr-preprod.spoc.pro.conf  /etc/nginx/sites-enabled/ 
 
 
 
5) redemarrer nginx
sudo service nginx restart
sudo service nginx  status : vérification ok

+ `PROSODY :`

 
1) Installer prosody : : Prosody trunk nightly build 718 (2016-10-18, 5022e6181193)
 
http://packages.prosody.im/debian/pool/main/p/prosody-trunk/prosody-trunk_1nightly718-1~yakkety_amd64.deb
sudo dpkg -i prosody-trunk_1nightly718-1~yakkety_amd64.deb
sudo apt-get install -f
 
2) Configurer prosody
 
/etc/prosody/prosody.cfg.lua : prosody.cfg.lua
 
/etc/prosody/conf.avail :
Création du répertoire si non nexistant : sudo mkdir conf.avail
 
/etc/prosody/conf.avail/ vtr-preprod.spoc.pro.cfg.lua
 
3) Activer la configuration prosody
 
sudo ln -s /etc/prosody/conf.avail/vtr-preprod.spoc.pro.cfg.lua  /etc/prosody/conf.d/
 
 
4) Génération Certificat
 
sudo apt install luarocks
sudo apt install lua5.1 liblua5.1-dev libidn11-dev libssl-dev
luarocks install luasec
 
sudo prosodyctl cert generate vtr-preprod.spoc
 
Key Size : 2048
 Leave the field empty to use the default value or '.' to exclude the field.
countryName (GB): FR
localityName (The Internet): Palaiseau
organizationName (Your Organisation): Immanens
organizationalUnitName (XMPP Department): SpocPro
commonName (jitsi.immanens.local): 
emailAddress (xmpp@jitsi.immanens.local): 
 
 
ssl = {
        certificate = "/var/lib/prosody/vtr-preprod.spoc.pro.crt";
        key = "/var/lib/prosody/vtr-preprod.spoc.pro.key";
 }
 
 
5) Génération de l'utilisateur 'focus'
 
sudo prosodyctl register focus auth.vtr-preprod.spoc.pro <secret>
  
 
6) Redemarrage du service
 
sudo prosodyctl restart
 
7) Vérification service opérationnel :
 
telnet localhost 5582
 
server:version() : vérification version prosody en cours
user:list("auth.vtr-preprod.spoc.pro")

+ `JITSI VIDEOBRIDGE :`
 
1 ) A partir des .deb fournit dans le zip Installer
sudo apt-get install default-jre-headless
dpkg -i jitsi-videobridge_829-1_amd64.deb
 
2) Configurer Videobridge
 
/etc/jitsi/videobridge/config : config
 
3) redemarrer le service et vérifier pas de pb dans les logs
sudo service jitsi-videobridge restart
 
 + `JICOFO :`
 
 1 ) A partir des .deb fournit dans le zip Installer jicofo
 
dpkg -i jicofo_1.0-299-1_amd64.deb
 
2) Configurer jicofo :
/etc/jitsi/jicofo/config : config
 
3) redemarrer le service et vérifier pas de pb dans les logs
 
sudo service jicofo restart

 + `JITSI-MEET -PROSODY :`
 
 dpkg -i jitsi-meet-prosody_1.0.1378-1_all.deb
 
  + `JITSI-MEET :`
  
  Recopier le source fourni dans le zip livré  (répertoire jitsi-meet) sous
le répertoir indiqué par la configuration nginx
(root /usr/share/jitsi-meet3/jitsi-meet/  dans mon fichier de configuration , mais à reconfigurer )
 
ne pas oublier de recopier et mettre à jour le fichier config.js en fonction du domaine  vtr-preprod.spoc.pro
 
Ensuite si des modifications de sources sont nécessiares , suivre la procédure indiquée dans mon précédent email :
installation npm , grunt  requise :
 
sudo apt-get install npm nodejs-legacy

  + `ScreenSharing :`
  
Extension Google chrome crée à partir de jidesha-master
L'extension actuellemnt publiée doit être mise à jour , son utilisation impliquera une modification du fichier de configuration config.js avec l'id correspondant
En attendant la version instance actuelle se base sur la version non empaquetée incluse dans le zip ci-joint :
-> google Chrome ->plus d outils -> extension -> mode developpeur -> charger l'extension non empaquetée -> pointer sur le répertoire chrome du zip extracté.
A noter : les deux extensions chrome sont quasiment identiques et autorisent le screensharing pour les 2 sites  vtr-preprod.spoc.pro et vtr.spoc.pro
par contre , l'installation automatique d 'unee xtension en peut se faire qu 'à partir d 'un site certifié par google et  un seul . Cela explique qu 'il y a duplication de l'extension pour la publication sur le chrome web store
A partir du moment qu 'une des 2 extensiosn installé , le screensharing sera opérationnel pour les 2 sites






