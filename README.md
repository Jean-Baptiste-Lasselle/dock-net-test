# Objet du repo

Ce petit repo pour répertorier les tests réseaux utilisés à la volée dans le projet CMS OneCloud Carrefour


### Principe Tests Réseau netcat/docker

Ensuite, étant donné 2 machines `M1` et `M2`, on peut mener le test suivant:

* sur  `M2`, on installe netcat, et on exécute: `nc -ul $NUMERO_PORT`, où `export NUMERO_PORT=514`. Alors, netcat attendra l'arrviée de paquets `UDP` par le port UDP numéro `514`, sur la machine `M2` 
* sur `M1`, on installe netcat, et on exécute: `nc -u $NOM_D_HOTE $NUMERO_PORT` , où `export NUMERO_PORT=514`, et où `$NOM_D_HOTE` est un nom d'hote réseau valide identifiant `M2` (au pire, `/etc/hosts` local). Alors, tous les paquets que nous enverrons avec netcat de cette manière seront reçus par le procsessus netcat s'exécutant sur `M2`.

installer netcat dans centos:

```bash
sudo yum install -y nc
```

Dans le paragrpahe ci-dessus, en retirant la lettre "u" de la liste des options d'invocation netcat, vous obtiendrez le même test, mais pour `TCP`, au lieu d'`UDP`.


## Exemples de tests effecutés chezs un de mes anciens clients

### Exemple Test Réseau 1 

Chez un de mes anciens clients, pour tester si le serveur Rsyslog reçoit bel et bien les paquets UDP envoyés par un client Rsyslog:

* Installez `netcat` sur le poste client concerné, sur le serveur Rsyslog, et le serveur ELK 
* Exécutez:

```bash
$ nc -v -u -z -w 3 cms.onecloud.carrefour.com 510-515
```

Ci-dessous, le résultat indique que sur la machine d'adresse IP 172.19.15.74, le port 514 est bien ouvert :

```bash
jb.lasselle@btek180330:~$ nc -v -u -z -w 3 rsyslog.bosstek.labs 510-515
rsyslog.bosstek.labs [172.19.15.74] 514 (syslog) open
```


### Exemple Test Réseau 2

Test pour savoir si le serveur  Rsyslog reçoit bel et bien les logs transmis par les clients Rsyslog:

* Installer netcat sur le Serveur Rsyslog, et exécutez : `nc -ul 514`, 
* Sur le client Rsyslog testé, faire une comande sudo et volontairement renseigner un mauvais mot de passe : `su jb.lasselle`

Exemple de résultat:

```bash
[root@server180627 jb.lasselle]# nc -ul 514
<85>Jul 20 17:33:02 btek180330 su[13711]: pam_krb5(su:auth): authentication failure; logname=jb.lasselle uid=1007 euid=0 tty=/dev/pts/2 ruser=jb.lasselle rhost=<85>Jul 20 17:33:02 btek180330 su[13711]: pam_unix(su:auth): authentication failure; logname= uid=1007 euid=0 tty=/dev/pts/2 ruser=jb.lasselle rhost=  user=jb.lasselle<85>Jul 20 17:33:02 btek180330 su[13711]: pam_sss(su:auth): authentication failure; logname= uid=1007 euid=0 tty=/dev/pts/2 ruser=jb.lasselle rhost= user=jb.lasselle<85>Jul 20 17:33:02 btek180330 su[13711]: pam_sss(su:auth): received for user jb.lasselle: 7 (Échec d'authentification)<85>Jul 20 17:33:04 btek180330 su[13711]: pam_authenticate: Authentication failure<85>Jul 20 17:33:04 btek180330 su[13711]: FAILED su for jb.lasselle by jb.lasselle<86>Jul 20 17:33:04 btek180330 su[13711]: - /dev/pts/2 jb.lasselle:jb.lasselle

```


### Exemple Test Réseau 3

Test pour savoir si le serveur  ELK reçoit bel et bien les logs transmis par le serveur Rsyslog:

* Installer netcat dans le conteneur Docker ELK:

```bash
export NOM_CONTENEUR_ELK=pile-elk-bosstek
sudo docker exec -it $NOM_CONTENEUR_ELK /bin/bash -c "apt-get install -y netcat"

```

* et exécutez : `nc -ul 514`, 

```bash
export NOM_CONTENEUR_ELK=pile-elk-bosstek
# il faut d'abord arrêter logstash, pour "prendre sa place dans le conteneur"
sudo docker exec -it $NOM_CONTENEUR_ELK /bin/bash -c "kill logstash"
sudo docker exec -it $NOM_CONTENEUR_ELK /bin/bash -c "nc -ul 514"

```
* Sur le client Rsyslog testé, faire une comande sudo et volontairement renseigner un mauvais mot de passe : `su jb.lasselle`

Expect: on s'attend à recevoir des paquets de la part du serveur Rsyslog, dans le conteneur destiné à logstash.

### Exemple Test Réseau 4

Même test que précédemment, sauf que le processus netcat attendant l'arrviée de paquets UDP de la part du serveur Rsyslog, est exécuté dans l'hôte Docker, au lieu du conteneur. On arrêtera alors tout simplement le conteneur, avant de lancer le processus netcat:

```bash
export NOM_CONTENEUR_ELK=pile-elk-bosstek
# Hôte Docker Cent OS 7
sudo yum install -y nc
sudo docker stop $NOM_CONTENEUR_ELK
sudo nc -ul -p 10514

```

# Architecture référencée dans 


## Schéma simplifié

Qui donne la vision simplifié de la dynamique des données transdférées et traitées:

![Architecture simplifiée](https://raw.githubusercontent.com/Jean-Baptiste-Lasselle/dock-net-test/master/documentation/architecture/solution-supervision-bosstek-archtiecture-recommandee.png)




## Schéma détaillé

Qui donne la vision pour l'orchestration des opérations de mise en service : 

![Architecture détaillée](https://raw.githubusercontent.com/Jean-Baptiste-Lasselle/dock-net-test/master/documentation/architecture/solution-supervision-bosstek-archtiecture-recommandee-NETWORK.png)



