# Guide d'Installation et de Configuration de Serveurs sur Debian

Ce guide vous explique comment installer et configurer un serveur DHCP, un serveur FTP/SFTP, et un serveur DNS sur deux machines virtuelles Debian sans interface graphique. Les machines seront configurées dans un réseau virtuel créé par votre logiciel de virtualisation.

## Étape 1 : Installation de Debian sans interface graphique

1. **Création des machines virtuelles** :
   - Créez deux machines virtuelles Debian sans interface graphique.
   - Configurez-les dans un réseau virtuel créé par votre logiciel de virtualisation (ex: VirtualBox, VMware, etc.).

## Étape 2 : Mise à jour des systèmes

1. **Mise à jour des paquets** :
   - Sur chaque machine, exécutez les commandes suivantes pour mettre à jour les paquets :
     ```bash
     sudo apt update
     sudo apt upgrade -y
     ```

## Étape 3 : Configuration du Serveur DHCP

1. **Installation du serveur DHCP** :
   - Sur la première machine, installez le serveur DHCP :
     ```bash
     sudo apt install isc-dhcp-server -y
     ```

2. **Configuration du serveur DHCP** :
   - Modifiez le fichier de configuration DHCP (`/etc/dhcp/dhcpd.conf`) pour attribuer des adresses de classe B :
     ```bash
     subnet 172.16.0.0 netmask 255.255.0.0 {
         range 172.16.0.2 172.16.255.254;
         option routers 172.16.0.1;
         option domain-name-servers 8.8.8.8, 8.8.4.4;
     }
     ```
   - Assurez-vous que la machine hébergeant le serveur DHCP possède une adresse IP fixe (ex: `172.16.0.1`).

3. **Redémarrage du service DHCP** :
   - Redémarrez le service DHCP pour appliquer les modifications :
     ```bash
     sudo systemctl restart isc-dhcp-server
     ```

## Étape 4 : Installation du Serveur FTP et SSH

1. **Installation de proFTPd et SSH** :
   - Sur la deuxième machine, installez proFTPd et SSH :
     ```bash
     sudo apt install proftpd openssh-server -y
     ```

2. **Configuration de proFTPd** :
   - Modifiez le fichier de configuration de proFTPd (`/etc/proftpd/proftpd.conf`) pour limiter les sessions à une seule connexion :
     ```bash
     MaxInstances 1
     ```
   - Configurez les identifiants FTP :
     - Identifiant : `laplateforme`
     - Mot de passe : `Marseille13!`

```bash
groupadd ftp2100
useradd -m -g ftp2100 -s /bin/bash laplateforme
passwd laplateforme
```

3. **Utilisation de SFTP** :
   - Utilisez le serveur SSH pour les connexions FTP en SFTP pour renforcer la sécurité.

## Étape 5 : Installation du Serveur DNS

1. **Installation du serveur DNS** :
   - Sur la première machine, installez le serveur DNS (ex: BIND9) :
     ```bash
     sudo apt install bind9 -y
     ```

2. **Configuration du DNS** :
   - Modifiez le fichier de configuration DNS (`/etc/bind/named.conf.local`) pour ajouter une zone :
     ```bash
     zone "ftp.com" {
         type master;
         file "/etc/bind/db.ftp.com";
     };
     ```
   - Créez le fichier de zone (`/etc/bind/db.ftp.com`) avec le contenu suivant :
     ```bash
     $TTL    604800
     @       IN      SOA     ns.ftp.com. admin.ftp.com. (
                             2023101001 ; Serial
                             604800     ; Refresh
                             86400      ; Retry
                             2419200    ; Expire
                             604800 )   ; Negative Cache TTL
     ;
     @       IN      NS      ns.ftp.com.
     ns      IN      A       172.16.0.2
     dns     IN      A       172.16.0.2
     ```
   - Redémarrez le service DNS :
     ```bash
     sudo systemctl restart bind9
     ```

## Étape 6 : Test de Connexion au Serveur SFTP

1. **Connexion au serveur SFTP** :
   - Utilisez un client SFTP (ex: FileZilla) pour vous connecter au serveur SFTP de la deuxième machine.
   - Utilisez l'adresse `dns.ftp.com` et les identifiants suivants :
     - Identifiant : `laplateforme`
     - Mot de passe : `Marseille13!`

## Conclusion

Vous avez maintenant configuré un serveur DHCP, un serveur FTP/SFTP, et un serveur DNS sur deux machines virtuelles Debian. Vous pouvez tester la connexion SFTP pour vérifier que tout fonctionne correctement.
