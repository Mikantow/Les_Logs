# Les_Logs  
## Challenge  
1 - Installe un serveur web (Apache ou Nginx) sur une machine virtuelle Linux  
2 - Configure le logging pour enregistrer les accès et les erreurs  
3 - Génère du trafic sur le serveur web (utilise des outils comme curl ou un navigateur)  
4 - Analyse les logs générés et identifie :  
  - Les requêtes réussies (code 200)  
  - Les erreurs 404 (page non trouvée)  
  - Les adresses IP les plus fréquentes

## Solution

1 Installation :

## Installation et configuration du service Apache2  

J'ai utilisé une VM Debian 12 sur virtual box avec un réseau en accès par pont.

Avant de commencer effectuer une mise à jour avec la commande suivante :

    sudo apt-get update && sudo apt-get upgrade -y  
    
Ensuite installer le servive apache 2 avec la commande suivante :  

    sudo apt-get install apache2 -y  

Afin de me connecter a mon serveur apache, je regarde mon ip avec la commande suivante :

    ip a  

Voici le résultat obtenu :

```bash
2: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:64:95:6a brd ff:ff:ff:ff:ff:ff
    inet 172.16.10.1/24 brd 172.16.10.255 scope global enp0s8
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe64:956a/64 scope link 
       valid_lft forever preferred_lft forever
```
Mon IP est donc 172.16.10.1/24

2 Configuration logging :

Pour modifier le logging il faudra modifier le fichier suivant :  

    /etc/apache2/apache2.conf  

3 Trafic :

Nous allons maintenant générer du trafic via notre VM principale en 172.16.10.1 et une seconde VM en 172.16.10.2 en passant par un navigateur web avec pour recherche l'ip du serveur apache2 (172.16.10.1)

4 Analyse des logs :

Suite au trafic effectuer, nous analysons les logs qui se trouve dans le fichier access.log, voici la commande a effectuer :

    cat /var/log/apache2/access.log  

Voici le résultat de la commande :

```bash
root@Debian12:/var/log/apache2# cat access.log 
172.16.10.1 - - [06/Jan/2025:10:35:11 +0100] "GET / HTTP/1.1" 200 3380 "-" "Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0"
172.16.10.1 - - [06/Jan/2025:10:35:11 +0100] "GET /icons/openlogo-75.png HTTP/1.1" 200 6040 "http://172.16.10.1/" "Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0"
172.16.10.1 - - [06/Jan/2025:10:35:12 +0100] "GET /favicon.ico HTTP/1.1" 404 489 "http://172.16.10.1/" "Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0"
172.16.10.2 - - [06/Jan/2025:11:18:43 +0100] "GET / HTTP/1.1" 200 3380 "-" "Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0"
172.16.10.2 - - [06/Jan/2025:11:18:46 +0100] "GET /favicon.ico HTTP/1.1" 404 490 "http://172.16.10.1/" "Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0"
172.16.10.2 - - [06/Jan/2025:11:18:46 +0100] "GET /icons/openlogo-75.png HTTP/1.1" 200 6040 "http://172.16.10.1/" "Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0"
172.16.10.2 - - [06/Jan/2025:11:18:55 +0100] "GET /net HTTP/1.1" 404 490 "-" "Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0"
172.16.10.2 - - [06/Jan/2025:11:19:08 +0100] "GET /toto HTTP/1.1" 404 490 "-" "Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0"
```

On remarque plusieurs trafic effectuer depuis deux adresses IP différente.

Nous allons maintenant filtrer les requêtes réussies et les erreurs avec la commande ``grep``

```bash
root@Debian12:/var/log/apache2# grep ' 200 ' /var/log/apache2/access.log                                
172.16.10.1 - - [06/Jan/2025:10:35:11 +0100] "GET / HTTP/1.1" 200 3380 "-" "Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0"
172.16.10.1 - - [06/Jan/2025:10:35:11 +0100] "GET /icons/openlogo-75.png HTTP/1.1" 200 6040 "http://172.16.10.1/" "Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0"
172.16.10.2 - - [06/Jan/2025:11:18:43 +0100] "GET / HTTP/1.1" 200 3380 "-" "Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0"
172.16.10.2 - - [06/Jan/2025:11:18:46 +0100] "GET /icons/openlogo-75.png HTTP/1.1" 200 6040 "http://172.16.10.1/" "Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0"
```

```bash
root@Debian12:/var/log/apache2# grep ' 404 ' /var/log/apache2/access.log
172.16.10.1 - - [06/Jan/2025:10:35:12 +0100] "GET /favicon.ico HTTP/1.1" 404 489 "http://172.16.10.1/" "Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0"
172.16.10.2 - - [06/Jan/2025:11:18:46 +0100] "GET /favicon.ico HTTP/1.1" 404 490 "http://172.16.10.1/" "Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0"
172.16.10.2 - - [06/Jan/2025:11:18:55 +0100] "GET /net HTTP/1.1" 404 490 "-" "Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0"
172.16.10.2 - - [06/Jan/2025:11:19:08 +0100] "GET /toto HTTP/1.1" 404 490 "-" "Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0"
```
  
