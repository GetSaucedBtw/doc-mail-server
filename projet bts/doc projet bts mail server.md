# ÉTABLIR UN SERVICE DE MESSAGERIE ÉLECTRONIQUE

Un **serveur de messagerie électronqiue** plus communément appelé serveur mail, consiste aux trasnferts de messages électroniques. Cependant, les **utilisateurs ne sont jamais en contact directement avec les serveurs** eux mêmes. L'envoie et la récepetion par l'utilisateur se fait à travers un **client de messagerie** installé sur son terminal (client lourd) ou alors sur une **messagerie web** (client léger).

## Plan Do Act Check (PDCA)
Dans un premier temps, nous verrons le **fonctionnement d'un serveur et ce qui en fait sa complexité**. Dans un deuxième temps, **les technologies et logiciels** que nous utiliserons. Dans un troisième temps, la **mise en place** des services vu au préalable. Dans un quatrième temps, nous ferrons alors les **vérifications du bon fonctionnement** du système de messagerie final.



## Fonctionnement d'un serveur de messagerie 
### Les agents

**MTA** de Mail Transfert Agent correspond au serveur de messagerie lui même. Le **MUA**, Mail User Agent, correspond au terminal depuis lequel l'utilisateur va pouvoir envoyer ou recevoir les courriers. 

### Notion de domaine
Le MTA étant le serveur mail, il est **dédié à une zone DNS** (Mail eXchange Records qui permet aux DNS de résoudre cette entrée en un serveur de messagerie). En effet, il gère alors un nom de domaine qui fait office de préfix au @ dans l'adresse électronnqiue qui se construit de cette maniere **agent/utilisateur@nom.de.domaine** De ce fait, la **gestion du serveur de noms est recommandée** afin de gérer au mieux ce serveur. 

#### MX Records et résolution DNS
Pour comprendre, le **MX Records** ne résoud pas un nom en addresse comme les entrées A ou AAAA. Pour ce faire, elle résoud vers un **FQDN (Full Qualified Domain Name)** c'est-à-dire que l'entrée renvoie vers le nom d'hôte du serveur de messagerie dédié au domaine. Ensuite, une résolution de type A ou AAAA est réalisé sur le FQDN obtenue grâce à la réponse précédente. Donc il faut 2 requêttes DNS afin de résourdre une envoi de courrier (La première pour récupérer le FQDN du serveur de messagerie et la seconde pour résoudre ce FQDN en adresse IP). Il est possible d'avoir **plusieurs serveurs de messagerie répondants à un domaine**, une **priorité est alors à définir** sous forme de chiffre dans l'entrée MX. Plus le **chiffre** de priorité est **faible** plus sa **priorité** en est **importante**.

**ATTENTION : LES REQUÊTTES DNS SONT EFFECTUÉES PAR LES MTA ET JAMAIS PAR LES MUA**


### Échange d'un courrier
Enfin pour **envoyer et recevoir un courrier**, il devra être **envoyé depuis un MUA** vers le premier **MTA**, cet envoie est **géré par le protocole SMTP**. Ensuite, ce **MTA le renvoie vers le MTA de réception** (MTA gérant la zone dédié dans l'adresse du recepteur), le protocole **SMTP** est réutiliser pour éffecteur ce transfert (Le protocole SMTP ne sert qu'à l'envoie MUA vers MTA ou alors MTA vers MTA). Enfin, la **réception s'effectue suivant les protocoles POP3 (télécharge les courriers localement) ou IMAP (synchronise les courriers sur le serveur)** après que le **MUA récepteur** ai fait la **demande au MTA récepteur**, qui peut ensuite lui **délivrer le courrier**.

<img src="../projet bts/captures mail/500px-Etapes_envoi_email.png" alt="schémad'organisation"> 






### La compléxité du service 
#### Le MTA
Comme expliqué au dessus pour **envoyer un courrier électronnique**, il faut alors **2 MTA**, un pour **transmettre** et l'autre pour **recevoir** et **redistribuer**. Cependant une **première difficulté** intervient ici, si **le serveur qui envoie se retrouve dans la position de récepteur** alors il faut qu'il soit en **capacité de pouvoir lui aussi recevoir et redistribuer**. Pour ce faire il faut alors comprendre que **les services d'envoie et de réception sont indépendant**. Il est possible alors de **faire un serveur dédié** à l'envoie et un serveur dédié à la réception bien que ce ne soit **pas la solution la plus optimisée**. Il faut alors prévoir de **monter deux services**, un pour **transmettre** et l'autre pour gérer la **réception et la distribution aux utilisateurs**.

#### Le MUA
De plus, nous ne parlons là que des **MTA**. Pour les **utilisateurs** il est alors **inconcevable** de devoir se **connecter directement sur le serveur** afin d'effectuer des actions, c'est pour ça que les **MUA** sont disponibles. Il faut alors aussi **s'organiser** pour choisir un **service de client** pour les courriers.

#### Une couche supplémentaire 
Enfin, on pourrait s'arrêté à ce stade de la conception. En suivant ces informations, nous serions en **capacité de construire un service de messagerie fonctionnelle**. Cependant, il serait alors **démuni de couche de sécurité**. En effet, la méthode du **"Phishing"**, Hameçonnage en français, est la **méthode la plus utilisée** par les pirates afin de pénetré dans un système d'information. La **compléxité** du système de messagerie est alors dû à **cette couche de sécurité** qui vient prendre une place considérable dans la conception. La **gestion** des **spams**, **virus** et de **l'authentification des domaines** est **primordiale** dans ce service. 


## Les services

### Postfix
Pour la partie du service **dédié au transfert de courrier (SMTP)**, nous utiliserons **Postfix**.C'est un **serveur de messagerie (MTA) pour les systèmes UNIX**.  De plus il permet une **inétgration à l'active directory grace au protocol LDAP** (Lightweight Directory Access Protocol) qui permet de **récupérer dynamiquement des informations**. 

<img src="../projet bts/captures mail/Postfix_Logo.svg.png" alt="Postfix Logo" width="50%">

### Dovecot
**Dovecot** est un **MTA Linux dédié à la relation avec les MUA** (POP3 et IMAP). Il laisse la possibilité de **choisir son client**, qu'il soit lourd (EX. Thunderbird) ou encore un client léger (Roundcube). Il englobe aussi une **intégration LDAP** qui permet alors de faciliter son **intégration avec Postfix**.

<img src="../projet bts/captures mail/dovecot.png" alt="Dovecot Logo" width="50%">

### Rspamd
Pour la partie **filtrage** du courrier on utilisera **Rspamd**. C'est un **moteur antispam et de filtrage** de contenu **rapide**. Il **analyse les courriers** entrants (peut aussi analyser les courriers sortants) et attribue un **score de spam** selon certains **critères** que nous pouvons définir. Il est alors en charge de faire un tri, ce qui lui octroie les **droits de rejeter** ceratin courriers, **marquer comme spam** ou juste d'ajouter un score. Il est doté d'une **interface web de management** qui simplifie son usage. 

<img src="../projet bts/captures mail/rspamd_logo_black.png" alt="Rspamd Logo" width="50%">

#### Les Modules
Rspamd permet d'intégré des **modules** au sein de l'outil. Ces modules sont des **services indépendant** qui peuvent lui être **rattachés**. Lorsque **Postfix** reçoit un courrier, il le **transmet à Rspamd** qui s'occupe de la couche de sécurité. Cette **sécurité** est alors encore divisé en **sous-couches** qui sont ces modules. **Chacun d'eux le traitent** en donnant un **score**, Rspamd fait ensuite un **verdict** de ce score ce qui lui permet de **faire une action** de rejet, désigner comme spam ou ne rien faire avan,t de le renvoyer à Potfix.

Nous privilégerons ceratins modules du fait de leur activité. **DKIM**, **DMARC**, **CLAMAV**, **BAYES** et **MIME_TYPES** sont des modules qui permettent de vérifier **l'authenticité des domaines** d'où proviennnt les courriers (DKIM et DMARC), CLAMAV est un **antivirus** qui scan les courrier afin d'en désceller un potentiel virus, BAYES est un système **d'apprentissage automatique** de détection de spam et MIME_TYPES permet de vérifier **l'authenticité des pièces jointes**.


## Mise en place

### Entrées DNS 
Il est préférable de **définir en amont** de l'installation, une entrée de type MX. Pour ce faire, nous utiliserons le service **Unbound DNS** intégré à notre pare-feu **OPNsense**. Il suffit alors de se rendre dans le menu **Services**, puis **Unbound DNS** et choisir **Overrides**. Une fois sur cette page, il faut **ajouter une entrée** et la définir comme **MX Records**, choisir le nom du serveur, sa correspondance en nom MX, le domain auquel il appartient et sa priorité (*La définir est obligatoire même si nous ne disposons que d'un serveur de messagerie*). 

<img src="../projet bts/captures mail/conf mx records.png" width="80%">

Enfin, Il faut alors créer une **entrée DNS de type A (IPv4)** afin de donnée une **correspondance au FDQN en adresse IP**.
Le procédé est le même, en choisissant type A.

<img src="../projet bts/captures mail/conf mail server dns a.png" width="80%">

### Postfix

#### Installation
Postfix est disponible depuis une installation à travers le dépôt d'installation Debian. Pour ce faire il faut alors taper ces commmandes :

```sudo apt update && sudo apt upgrade -y``` 

*Ces commandes permettant alors de pouvoir vérifier si une mise à jour des paquets du repertoire de dépôt est disponible et si nécessaire de les appliquer*

Enfin il est alors possible de l'installer à l'aide de 

```sudo apt install postfix -y```

<img src="../projet bts/captures mail/postfix install.png" width="75%">

Une Page d'installation Debian va alors appraitre nous permettant de sélectionner le type de serveur de messagerie nous voulons installer. Ils nous est disposé de 5 options, nous prendrons la 2e correspondant à **Internet Site**. Cette option défini alors que les courriers sont envoyés et reçus en utilisant le protocole **SMTP**. Ensuite, ils nous faut alors définri le **FQDN** (*doit être identique à celui déclaré dans les entrées DNS*) du serveur mail, dans notre cas nous utiliserons **mail.cdg.sio.pub**. Une fois choisi le serveur s'installera.


#### Configuration
Une fois installé, la configuration du service Postfix se fait dans le fichier *main.cf* à l'emplacement */etc/postfix/main.cf*. Dans ce fichier sont présents les chemins indiquant les certificats ou les clés d'authentification du serveur. Il permet aussi de définir le FQDN du serveur, les réseaux sur lesquels sont présent des relais (*si plusieurs serveurs de messagerie sont déclarés pour le même domaine*). Dans notre configuration, nous définirons dans un premier temps le domaine sur lequel il execercera, les destinations qu'il considère comme lui-même et nous vérifirons son hostname. Pour accéder au fichier : 

```cd /etc/postfix && sudo nano main.cf```

Pour configurer le domaine, il faut modifier le premier paragraphe situé sous l'indication de la version de Postfix. Dans notre cas, nous définirons notre domaine par cdg.sio.pub et changerons *myorigin* par *$mydomain* (changer la définition de myorigin permet si un compte local envoie un mail d'avoir comme domain cdg.sio.pub et non mail.cdg.sio.pub)

<img src="../projet bts/captures mail/domain postfix.png" widht="80%">

Ensuite, on peut alors définir ses destinations qu'il considère comme lui-même. Cette option se démarque à l'aide de la ligne *mydestination*. On indiquera alors quels hosts lui correspondent sous différentes manières (loopback, loopback.domain.name, FQDN et sous la fonction $myhostname)

```mydestination = $myhostname, mail.cdg.sio.pub, localhost, localhost.cdg.sio.pub```

Enfin, on peut retrouver l'hostname dans les dernières ligne du fichier. On vérifie alors qu'il correspond à celui définit lors de l'installation qui doit être le même rensaigné dans les entrées DNS.

<img src="../projet bts/captures mail/postfix hostname.png">

Une fois modifié on peut alors auvegarder le fichier et redémarrer Postifx à l'aide de cette commande : 

```sudo systemctl restart postfix.service && sudo systemctl status postfix.service```

### Dovecot

#### Installation
Pour l'intallation de Dovecot, ils prévoient des paquets dans les dépôts Debian. À l'aide de leur documentation, on peut alors vérifier lesquels sont adaptés pour notre usage. Nous installerons alors *dovecot-core* (système usuel), *dovecot-po3d* (le daemon POP3), *dovecot-imapd* (le daemon IMAP), *dovecot-lmtpd* (le daemon LMTP (Light Mail Transfert Protocol) qui permet d'échanger des mail comme le SMTP mais en local (liaison avec Postfix)) et *dovecot-ldap* (gère l'intégration LDAP). 

```sudo apt install dovecot-core dovecot-ldap dovecot-pop3d dovecot-imapd dovecot-lmtp -y```

*SI VOTRE DEBIAN 13 NE TROUVE PAS LES PAQUETS CORRESPONDANTS A DOVECOT VOUS POUVEZ LES AJOUTER AUX DÉPÔTS SOURCES EN SUIVANT CES INSTRCUTIONS [Ajout des dépôts Dovecot](https://repo.dovecot.org/)*

#### Configuration
Tous les fichiers de configuration liés à Dovecot se situe dans ce répertoire : */etc/dovecot*
Dans ce répertoire, nous trouverons alors le fichier **dovecot.conf**. Par défaut, toute la configuration est commenté à l'aide de #, il nous faut alors décommenter la ligne correspondante à ```protocols = imap pop3 lmtp```.

<img src="../projet bts/captures mail/proto dovecot.png" wdith="80%">

