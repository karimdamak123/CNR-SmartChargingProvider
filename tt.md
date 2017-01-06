# installation  jitsi version 2.9 

###`Introduction :`

Packages JITSI à installer  (uniquement ceux -la)<br/>
jitsi-videobridge<br/>
jicofo<br/>
jitsi-meet-prosody<br/>
jitsi-meet<br/>
 
Installation : (ordre à respecter)

###`OS :`

WM os linux VM locale Ubuntu 16.10

###`Java :`

openjdk version "1.8.0_102"<br/>
sudo apt-get install openjdk-8-jdk<br/>
sudo apt-get install openjdk-8-jre-headless<br/>

###`php :`

sudo apt install php7.0-cli<br/>
sudo apt-get -y install php7.0-fpm<br/>
modifier le fichier /etc/php/7.0/fpm/php.ini<br/>
cgi.fix_pathinfo=0   (enlever le commentaire « ; » et le mettre à 0)<br/>
puis redémarrer:<br/>
sudo systemctl restart php7.0-fpm<br/>
curl: sudo apt-get install php-curl<br/>

###`NGINX :`

1) Installer  nginx
sudo apt-get install nginx<br/>
Lanceur dans /etc/init.d/nginx.<br/>
Binaire dans /usr/sbin/nginx.<br/>
Pid dans /run/nginx.pid.<br/>
Defaults dans /etc/default/nginx.<br/>
Configuration dans /etc/nginx/.<br/>
Logs dans /var/log/nginx/.<br/>

2 ) Génération d'un certificat SSL (avec  l’outil Cerbot)
cd /usr/local/sbin<br/>
sudo wget https://dl.eff.org/certbot-auto<br/>
sudo chmod a+x /usr/local/sbin/certbot-auto<br/>
sudo ./certbot-auto certonly -a webroot --webroot-path=/usr/share/jitsi/jitsi-meet/ -d vtr-preprod.spoc.pro<br/>
puis ajouter dans le fichier /etc/nginx/sites-available/vtr-preprod.spoc.pro.conf :<br/>
     ssl_certificate /etc/letsencrypt/live/vtr-preprod.spoc.pro/cert.pem; <br/>
     ssl_certificate_key /etc/letsencrypt/live/vtr-preprod.spoc.pro/privkey.pem; <br/>

enfin ajouter dans le cron : <br/>
sudo crontab -e <br/>
30 2 * * 1 /usr/local/sbin/certbot-auto renew >> /var/log/le-renew.log <br/>
35 2 * * 1 /etc/init.d/nginx reload <br/>
  
3) Configurer nginx
/etc/nginx/sites-available : vtr-preprod.spoc.pro.conf
 
4) Activer la configuration
sudo ln -s /etc/nginx/sites-available/vtr-preprod.spoc.pro.conf  /etc/nginx/sites-enabled/ 
 
5) redemarrer nginx
sudo service nginx restart<br/>
sudo service nginx  status : vérification ok <br/>

###`PROSODY :`
 
1) Installer prosody : : Prosody trunk nightly build 718 (2016-10-18, 5022e6181193)
 
http://packages.prosody.im/debian/pool/main/p/prosody-trunk/prosody-trunk_1nightly718-1~yakkety_amd64.deb<br/>
sudo dpkg -i prosody-trunk_1nightly718-1~yakkety_amd64.deb <br/>
sudo apt-get install -f <br/>
 
2) Configurer prosody
 
/etc/prosody/prosody.cfg.lua : prosody.cfg.lua<br/>
 
/etc/prosody/conf.avail :<br/>
Création du répertoire si non nexistant : sudo mkdir conf.avail<br/>
 
/etc/prosody/conf.avail/ vtr-preprod.spoc.pro.cfg.lua<br/>
 
3) Activer la configuration prosody
 
sudo ln -s /etc/prosody/conf.avail/vtr-preprod.spoc.pro.cfg.lua  /etc/prosody/conf.d/<br/>
 
 
4) Génération Certificat
 
sudo apt install luarocks<br/>
sudo apt install lua5.1 liblua5.1-dev libidn11-dev libssl-dev<br/>
luarocks install luasec<br/>
 
sudo prosodyctl cert generate vtr-preprod.spoc<br/>
 
Key Size : 2048<br/>
 Leave the field empty to use the default value or '.' to exclude the field.<br/>
countryName (GB): FR<br/>
localityName (The Internet): Palaiseau<br/>
organizationName (Your Organisation): Immanens<br/>
organizationalUnitName (XMPP Department): SpocPro<br/>
commonName (jitsi.immanens.local): <br/>
emailAddress (xmpp@jitsi.immanens.local): <br/>
 
 
ssl = {<br/>
        certificate = "/var/lib/prosody/vtr-preprod.spoc.pro.crt";<br/>
        key = "/var/lib/prosody/vtr-preprod.spoc.pro.key";<br/>
 }
 
 
5) Génération de l'utilisateur 'focus'
 
sudo prosodyctl register focus auth.vtr-preprod.spoc.pro <secret><br/>
  
 
6) Redemarrage du service
 
sudo prosodyctl restart<br/>
 
7) Vérification service opérationnel :
 
telnet localhost 5582<br/>
 
server:version() : vérification version prosody en cours<br/>
user:list("auth.vtr-preprod.spoc.pro")<br/>

###`JITSI VIDEOBRIDGE :`
 
1 ) A partir des .deb fournit dans le zip Installer
sudo apt-get install default-jre-headless<br/>
dpkg -i jitsi-videobridge_829-1_amd64.deb<br/>
 
2) Configurer Videobridge
 
/etc/jitsi/videobridge/config : config<br/>
 
3) redemarrer le service et vérifier pas de pb dans les logs
sudo service jitsi-videobridge restart<br/>
 
###`JICOFO :`
 
 1 ) A partir des .deb fournit dans le zip Installer jicofo
 
dpkg -i jicofo_1.0-299-1_amd64.deb<br/>
 
2) Configurer jicofo :
/etc/jitsi/jicofo/config : config
 
3) redemarrer le service et vérifier pas de pb dans les logs
 
sudo service jicofo restart

###`JITSI-MEET -PROSODY :`
 
 dpkg -i jitsi-meet-prosody_1.0.1378-1_all.deb
 
###`JITSI-MEET :`
  
  Recopier le source fourni dans le zip livré  (répertoire jitsi-meet) sous
le répertoir indiqué par la configuration nginx
(root /usr/share/jitsi-meet3/jitsi-meet/  dans mon fichier de configuration , mais à reconfigurer )
 
ne pas oublier de recopier et mettre à jour le fichier config.js en fonction du domaine  vtr-preprod.spoc.pro
 
Ensuite si des modifications de sources sont nécessiares , suivre la procédure indiquée dans mon précédent email :
installation npm , grunt  requise :<br/>
 
sudo apt-get install npm nodejs-legacy

###`ScreenSharing :`
  
Extension Google chrome crée à partir de jidesha-master
L'extension actuellemnt publiée doit être mise à jour , son utilisation impliquera une modification du fichier de configuration config.js avec l'id correspondant<br/>
En attendant la version instance actuelle se base sur la version non empaquetée incluse dans le zip (chrome_jidesha.zip) :
-> google Chrome ->plus d outils -> extension -> mode developpeur -> charger l'extension non empaquetée -> pointer sur le répertoire chrome du zip extracté.<br/>
A noter : les deux extensions chrome sont quasiment identiques et autorisent le screensharing pour les 2 sites  vtr-preprod.spoc.pro et vtr.spoc.pro<br/>
par contre , l'installation automatique d 'unee xtension en peut se faire qu 'à partir d 'un site certifié par google et  un seul . Cela explique qu 'il y a duplication de l'extension pour la publication sur le chrome web store
A partir du moment qu 'une des 2 extensiosn installé , le screensharing sera opérationnel pour les 2 sites






