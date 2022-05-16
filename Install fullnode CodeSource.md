
# Installation d'un fullnode APTOS pour le Devnet avec le code source

Organisation

4 fenêtres à ouvrir pour effectuer cette installation

1) La fenêtre qui contient la doc que je vous livre
2) Les 2 fenêtres de terminal ubuntu pour effectuer l'installation.
3) Une page web à l’url https://aptos-node.info


C'est parti :

Pré requis :

Tout d'abord les pré-requis :

Hardware :

Un serveur avec minimum 2 cœurs et 4Gb de mémoire mais 4 cœurs et 8Gb de mémoire sont préférables

Pour connaitre ce que votre serveur a sous le capot utilisez les commandes suivantes :

```bash
lsmem 
cat /proc/cpuinfo | grep processor | wc -l
```


##Avoir un serveur ou Aptos est complètement désinstallé et plus aucun process Aptos qui tourne 
##Ou alors c'est une première install donc là pas de souci

Connaitre son ip publique et son ip "interne"

Pour avoir son ip interne via la commande :

```bash
ifconfig –a
```

L'adresse est celle en 192....


Pour avoir son ip publique tapez la commande :

```bash
wget -qO- icanhazip.com
```

Gardez au chaud ces deux informations.


Ensuite les ouvertures de port :

Il faut ouvrir les ports 6180, 9101 et 8080 sur votre serveur mais aussi sur votre votre box du fournisseur internet
Pour ouvrir sur votre serveur voici la commande :

```bash
ufw enable
ufw allow 6180
ufw  allow 9101
ufw  allow 8080
```

Sur votre box c'est à vous de voir car chaque box est différente. Chez un hébergeur en VPS en général les ports sont ouverts (juste la partie serveur à faire)



C'est parti :

L'installation se fait en 3 étapes :

1- L'install du "moteur" Aptos
2- La création de nos clefs privées et publiques
3- L'upgrade pour etre en phase avec les dernières releases Aptos


Etape 1 :

Installation du moteur :

Se connecter en tant que root
Si vous utilisez un autre user taper la commande :

```bash
sudo su -
```

Aller dans /root via la commande :

```bash
cd /root
```

Downloader le répertoire aptos-core à partir du Github:

Pour cela vous devez d’abord vous créer un compte sur Github :

Se connecter sur https://github.com

Suivre la procédure (sign in pour se créer un compte)

Gardez votre pseudo de connexion car il est maintenant nécessaire pour charger l’aptos-core.

Aller sur la page suivante :

https://github.com/aptos-labs/aptos-core

En haut à droite de cette page il y a un onglet fork, cliquez dessus.
Maintenant vous pouvez charger l’aptos-core via la commande suivante :

git clone https://github.com/<votre user de connexion github>/aptos-core
Une fois terminée vous avez un répertoire aptos-core sous /root
Aller dans ce répertoire via la commande :

```bash
cd /root/aptos-core
```

Puis lancer le script d'installation du moteur Aptos via la commande :

```bash
./scripts/dev_setup.sh
```

Charger l'environnement cargo via la commande :

```bash
source ~/.cargo/env
```

Mettre à jour la branche pour le devnet :

```bash
git checkout origin/devnet
```

Le moteur aptos-core contient un fichier pré-rempli nommé public_full_node.yaml
Ce fichier c'est la conf de votre fullnode et il est extrêmement sensible

Copiez le fichier mis à disposition dans /root/aptos-core via la commande:

```bash
cp config/src/config/test_data/public_full_node.yaml /root/aptos-core/
```

Vérifiez que la copie est bien faite via la commande :

```bash
ls -la | grep public_full_node.yaml
```

Vous devriez voir le nom du fichier

Downloader les fichiers genesis.blob et waypoint.txt pour le devnet  (ces fichiers seront amenés à changer au fil du temps)

https://devnet.aptoslabs.com/genesis.blob 
https://devnet.aptoslabs.com/waypoint.txt

A noter le fichier waypoint.txt n'arrive pas sous forme de fichier mais juste des chiffres et lettres sur l'écran (à copier et garder pour un peu plus loin)

Créer les répertoires suivant sous /opt :

```bash
mkdir /opt/aptos
mkdir /opt/aptos/
mkdir /opt/aptos/data
mkdir /opt/aptos/etc
```

Le fichier genesis doit être copiés sous /opt/aptos/etc et le waypoint.txt est à créer au même endroit

Mon fichier genesis.blob téléchargé est sous : /home/kbconsulting45/Téléchargements je prendrai cela comme base à vous de voir ou sont les votres..

Copie du genesis.blob sous /opt/aptos/etc via la commande :

```bash
cp /home/kbconsulting45/Téléchargements/genesis.blob /opt/aptos/etc/
```


Création du fichier waypoint.txt sous /opt/aptos/etc/ via la commande :

```bash
sudo nano waypoint.txt
```

Copier le contenu gardé plus haut dans le fichier

Pour quitter le fichier et sauvegarder appuyer sur les touches control et x, puis O et enter


Maintenant éditer le fichier de configuration du fullnode :

```bash
cd /root/aptos-core
sudo nano public_full_node.yaml
```

Modifiez les lignes comme suit :

```bash
base:
    # Update this value to the location you want the node to store its database
    data_dir: "/opt/aptos/data"
    role: "full_node"
    waypoint:
        # Update this value to that which the blockchain publicly provides. Please regard the directions
        # below on how to safely manage your genesis_file_location with respect to the waypoint.
        from_file: "/opt/aptos/etc/waypoint.txt"

state_sync:
    state_sync_driver:
        enable_state_sync_v2: true

execution:
    # Update this to the location to where the genesis.blob is stored, prefer fullpaths
    # Note, this must be paired with a waypoint. If you update your waypoint without a
    # corresponding genesis, the file location should be an empty path.
    genesis_file_location: "/opt/aptos/etc/genesis.blob"

full_node_networks:
    - discovery_method: "onchain"
      # The network must have a listen address to specify protocols. This runs it locally to
      # prevent remote, incoming connections.
      listen_address: "/ip4/192.168.1.190/tcp/6180"
      network_id: "public"
      # Define the upstream peers to connect to
      seeds:
      {}
      
api:
    enabled: true
    address: 192.168.1.190:8080
```
Sortez du mode nano en appuyant sur Control et x, puis O et enter

Remarque : 
J'ai beaucoup galéré car je mettais comme indiqué dans le doc 0.0.0.1 ou 127.0.0.1 (qui correspond à mon ip ou localhost) et ça ne marchait pas
J'ai donc décidé de remplacer par l'IP interne et là ça a marché. Quand on est chez un hébergeur il n'y a pas de souci mais chez soit avec le fournisseur internet c'est parfois une autre histoire.

Vous avez donc votre fichier de configuration qui vous permet de lancer le fullnode dans sa configuration la plus simple :


Lancer la compilation du fullnode via la commande suivante :

```bash
cargo run -p aptos-node --release -- -f ./public_full_node.yaml
```

Cela peut durer jusqu’à 1h suivant votre pc.

Remarque : il n'est pas lancé en tache de fond et donc vous allez voir défiler les logs en permanence (pour l'instant ça suffira)

Laisser tourner sur ce terminal et allons vérifier que tout est ok :

Sur un autre terminal vérifions que la base de données est bien créée :

Pour se faire tapez la commande suivante :

```bash
ls -la /opt/aptos/data
```

Résultat :

drwxr-xr-x 3 root root 4096 mai    4 14:11 .
drwxr-xr-x 4 root root 4096 mai    4 13:58 ..
drwxr-xr-x 3 root root 4096 mai    4 14:11 db

Si vous voyez un répertoire db c'est gagné sinon vous avez loupé un truc dans la procédure et la copie de la blockchain n’est pas installé

Seconde vérification tapez la commande suivante :

```bash
curl 127.0.0.1:9101/metrics 2> /dev/null | grep "aptos_state_sync_version{.*\"synced\"}" | awk '{print $2}'
```

Si cela renvoie un nombre style 65700 il faut le comparer avec le nombre affiché sur cette page : https://status.devnet.aptos.dev/
Si le nombre affiché par la commande curl correspond au nombre affiché sur l’url vous etes synchronisés avec la blockchain Aptos. Bravo

Enfin si la commande suivante renvoie un nombre supérieur à zéro tout est ok :

```bash
curl 127.0.0.1:9101/metrics 2> /dev/null | grep "aptos_connections{direction=\"outbound\""
```

Dernière vérification Ouvrez une page web à l'url suivante :
https://node.aptos.zvalid.com

Si tout est ok tous les voyants sont au vert sur l’écran

Retournons sur le terminal ou la commande cargo a été lancée.
Le problème c’est que lorsque vous vous déconnectez le moteur s’arrête.
Pour ne pas avoir ce problème, il faut lancer la commande en tache de fond et en nohup.

Control C pour arreter le process en cours

Et relancer via la commande :

```bash
Nohup cargo run -p aptos-node --release -- -f ./public_full_node.yaml &
```

Là vous pouvez vous déconnectez et le nœud reste up.

----------------------------------------------
Maintenant déclarons ce service pour son lancement et son arrêt car on ne va pas recompiler à chaque fois que l'on veut arrêter ou démarrer le fullnode!!
Il va falloir créer ce fichier que nous nommerons aptos-node.service:
Pour cela taper les commandes suivantes :

```bash
sudo nano /etc/systemd/system/aptos-node.service
```

Entrer les lignes suivantes :

```bash
[Unit]
  Description=aptos-node daemon
  After=network-online.target
[Service]
  User=$USER
  ExecStart=/root/aptos-core/target/release/aptos-node --config /root/aptos-core/public_full_node.yaml
  Restart=on-failure
  RestartSec=5
  LimitNOFILE=4096
[Install]
  WantedBy=multi-user.target

```
Sortez via Control X, O puis enter

Permettons maintenant la prise en main du démarrage et arrêt par notre systeme :

```bash
sudo systemctl enable aptos-node.service
sudo systemctl daemon-reload
```

Ainsi pour la suite si démarrage du service :

```bash
sudo systemctl start aptos-node.service
```

Arret du service :

```bash
sudo systemctl stop aptos-node.service
```

Si cela ne fonctionne pas le plus simple c’est d’utiliser cette commande pour démarrer le nœud en tache de fond :

```bash
Nohup /root/aptos-core/target/release/aptos-node --config /root/aptos-core/public_full_node.yaml &
```
Ou
```bash
Nohup cargo run -p aptos-node --release -- -f ./public_full_node.yaml &
```

Et verification des logs :

```bash
journalctl -u aptos-node.service -f
```


Nous avons fini la première étape d'installation.


Deuxième Etape :


Maintenant la deuxième étape qui consiste à avoir une identité statique définie et stable pour notre fullnode :

Arrêter le fullnode via la commande suivante :

```bash
sudo systemctl stop aptos-node.service
```

Aller dans le répertoire aptos-core via la commande :

```bash
cd /root/aptos-core
```

Créer un répertoire identity via la commande :

```bash
mkdir identity
```

Puis lancer la commande suivante pour générer votre clef privée :

```bash
cargo run -p aptos-operational-tool -- generate-key --encoding hex --key-type x25519 --key-file /root/aptos-core/identity/private-key.txt
```

Vérifier que le fichier a bien été créé via la commande :

```bash
cat /root/aptos-core/identity/private-key.txt 
```

(C'est votre clef privée à ne révéler à personne, là on est en devnet ça va mais après c'est chasse gardée!!)



Puis générez votre clef publique à partir de votre clef privée via la commande :

```bash
cargo run -p aptos-operational-tool -- extract-peer-from-file --encoding hex --key-file /root/aptos-core/identity/private-key.txt --output-file /root/aptos-core/identity/peer-info.yaml
```

Vérifier que le fichier a bien été généré via la commande :

```bash
cat /root/aptos-core/identity/peer-info.yaml 
```

(vous avez là votre clef publique)

Remarque : ces deux informations vont devoir être intégrées au fichier de configuration du fullnode public_full_node.yaml


Editer le fichier public_full_node.yaml comme suit via la commande :

```bash
sudo nano public_full_node.yaml
```

Fichier de conf yaml

```bash
base:
    # Update this value to the location you want the node to store its database
    data_dir: "/opt/aptos/data"
    role: "full_node"
    waypoint:
        # Update this value to that which the blockchain publicly provides. Please regard the directions
        # below on how to safely manage your genesis_file_location with respect to the waypoint.
        from_file: "/opt/aptos/etc/waypoint.txt"

state_sync:
    state_sync_driver:
        enable_state_sync_v2: true

execution:
    # Update this to the location to where the genesis.blob is stored, prefer fullpaths
    # Note, this must be paired with a waypoint. If you update your waypoint without a
    # corresponding genesis, the file location should be an empty path.
    genesis_file_location: "/opt/aptos/etc/genesis.blob"

full_node_networks:
    - discovery_method: "onchain"
      # The network must have a listen address to specify protocols. This runs it locally to
      # prevent remote, incoming connections.
      listen_address: "/ip4/192.168.1.190/tcp/6180"
      network_id: "public"
      identity:
       type: "from_config"
       key: cequifiguredanslefichierprivatekey
       peer_id: lapremièresuitedenombredufichier
      # Define the upstream peers to connect to
      seeds:
       9f4b240926582a25aa595d14c033a8264c18b626feebe21d5402bd60b8319d2a:
          addresses:
          - "/ip4/161.35.81.173/tcp/6180/ln-noise-ik/9f4b240926582a25aa595d14c033a8264c18b626feebe21d5402bd60b8319d2a/ln-handshake/0"
          role: "Upstream"

#      48363b7627e0ff63b3e72bfc9b12c12b12ddba9dd1b6bdd0fbc77b433f95d90a:
#          addresses:
#          - "/ip4/157.245.65.169/tcp/6180/ln-noise-ik/48363b7627e0ff63b3e72bfc9b12c12b12ddba9dd1b6bdd0fbc77b433f95d90a/ln-handshake/0" 
#          role: "Upstream" 
#        {}

api:
    enabled: true
    address: 192.168.1.190:8080


```
Vous sortez du fichier via control X et O puis enter

et vous lancez à nouveau le fullnode via la commande :

```bash
sudo systemctl start aptos-node.service
```

Votre nœud est prêt avec une adresse publique fixe qui vous permet de réaliser des tx avec la blockchain

Idem vous pouvez taper les mêmes commandes de vérification qu'à l'étape 1 et aller sur l'url pour voir si tout est ok


Troisième étape

Etape 3 les mises à jour

Il est possible que même à la première installation via l'url aptos-node.info vous ayez le nœud pas à jour

Tout d'abord arrêter le fullnode via les commandes :

```bash
ps -ef | grep aptos
kill -9 ....
```

Ou 
```bash
sudo systemctl stop aptos-node.service
```

Il faut maintenant supprimer la base de données pour cela :

```bash
rmdir /opt/aptos/data
```

Ensuite il faut supprimer le genesis.blob 

```bash
rm /opt/aptos/etc/genesis.blob
```

Remettre la bonne branche du devnet :

```bash
git checkout origin/devnet
```

Puis charger les fichiers genesis.blob et waypoint.txt

https://devnet.aptoslabs.com/genesis.blob

https://devnet.aptoslabs.com/waypoint.txt  (idem ça n'est pas un fichier mais une ligne avec des chiffre lettre...à copier

Comme lors de l'étape 1 copier le fichier genesis.blob sous /opt/aptos/etc et éditer un fichier waypoint.txt en y insérant les données du fichier téléchargé

Editer le fichier  public_full_node.yaml  (si cela est nécessaire par exemple dernière le mode de synchro a changé et des lignes étaient à ajouter...)

Recompilez le noyau :

```bash
cargo run -p aptos-node --release -- -f ./public_full_node.yaml
```

Faites les vérifications pour voir si tout est ok en laissant tourner le moteur aptos.

Si tout est ok, appuyer sur Control C dans la fenêtre d’installation

Redémarrer le fullnode et c'est bon vous êtes à jour

```bash
sudo systemctl start aptos-node.service
```

Voili voilou j'espère que ce guide vous permettra de moins vous arracher les cheveux que moi au début 


