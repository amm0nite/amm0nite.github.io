---
layout: post
title: "Serveur samba simple"
published: true
categories: self-hosting
---

Samba est un logiciel GNU/Linux qui implémente le protocole partage de fichiers sur le réseau de Windows.

Je travaille et joue sur un PC sous Windows, depuis toujours je gardais tout mes fichiers sur un gros disque dur interne. Mais j'ai décidé de déplacer toutes mes données non temporaires sur mon serveur Debian, dans le but de les sécuriser ( En faisant des synchronisation entre mes serveurs, mais ce n'est pas le sujet ) et de les partager sur mon réseau local.

Windows est nul pour monter des systèmes de fichiers réseau basés sur FTP, NFS, SFTP, WebDAV… La meilleure intégration dans Windows est obtenue avec un partage propulsé par Samba. Voici la configuration que j'ai utilisée pour mettre en place un partage sans authentification avec accès réservé aux membres du réseau local. Le but n'est pas de tout expliquer mais juste de partager le résultat de mon travail.

@/etc/samba/smb.conf@

{% highlight ini %}

[global]
workgroup = WORKGROUP
netbios name = partagebox
server string = %h server

wins support = no
dns proxy = no

interfaces = eth0
bind interfaces only = yes
hosts allow = 192.168.0.0/24
hosts deny = 0.0.0.0/0

log file = /var/log/samba/log.%m
max log size = 1000

security = user
map to guest = bad user
guest account = nobody

[partage]
comment = mon dossier de partage
path = /chemin/du/dossier/partage/
browsable = yes
writable = yes
guest ok = yes

{% endhighlight %}

Ajouter l'utilisateur anonyme dans samba ( l'utilisateur Linux correspondant devra avoir des droits d'accès suffisant sur les fichiers ) : @smbpasswd -an nobody@

Une fois configuré le partage apparaît tout seul sur l'explorateur Windows et ne demande pas de nom d'utilisateur ni de mot de passe pour être ouvert.

[Explication de la configuration de l'authentification anonyme Windows friendly][mylink]

[mylink]:http://micheljansen.org/blog/entry/182