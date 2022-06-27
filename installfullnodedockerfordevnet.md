# Guide d'installation d'un fullnode en docker pour le devnet
# Comprend l'update du fullnode à chaque mise à jour du Devnet
# Par Kbconsulting45


# PARTIE 1 : INSTALLATION DU FULLNODE

# Pour éviter les problèmes, tout d'abord ouvrir tous les ports necessaires via la commande ufw :

```bash
ufw enable
ufw allow 22
ufw allow 6180
ufw allow 6181
ufw allow 6182
ufw allow 9101
ufw allow 8080
```

# Déclaration des variables:

```bash
echo "export WORKSPACE=devnet" >> $HOME/.bash_profile

echo "export PUBLIC_IP=$(curl -s ifconfig.me)" >> $HOME/.bash_profile

source $HOME/.bash_profile
```

# Update des Packages

```bash
sudo apt update && sudo apt upgrade -y
```

# Installation des dépendances :

```bash
sudo apt-get install jq unzip -y
```

# Installation de docker :

```bash
sudo apt-get install ca-certificates curl gnupg lsb-release -y

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt-get update

sudo apt-get install docker-ce docker-ce-cli containerd.io -y

```




# Installation du docker compose :

```bash
mkdir -p ~/.docker/cli-plugins/

curl -SL https://github.com/docker/compose/releases/download/v2.2.3/docker-compose-linux-x86_64 -o ~/.docker/cli-plugins/docker-compose

chmod +x ~/.docker/cli-plugins/docker-compose

sudo chown $USER /var/run/docker.sock
```

# Installation Aptos CLI :

```bash
wget -qO aptos-cli.zip https://github.com/aptos-labs/aptos-core/releases/download/aptos-cli-v0.1.1/aptos-cli-0.1.1-Ubuntu-x86_64.zip

unzip -o aptos-cli.zip -d /usr/local/bin

chmod +x /usr/local/bin/aptos

rm aptos-cli.zip
```

# Création du repertoire pour le fullnode :

```bash
mkdir /root/devnet
cd /root/devnet
```

# Chargement des fichiers de configuration .yaml :

```bash
wget https://raw.githubusercontent.com/aptos-labs/aptos-core/main/docker/compose/public_full_node/docker-compose.yaml
wget https://raw.githubusercontent.com/aptos-labs/aptos-core/main/docker/compose/public_full_node/public_full_node.yaml
```



# Edition des deux fichiers de configuration comme ci-dessous :
# D'abord le fichier docker-compose.yaml :

```bash
# This compose file defines a Public Fullnode docker compose wrapper around aptos-node.
#
# In order to use, place a copy of the proper genesis.blob and waypoint.txt in this directory.
#
# Developer testnet genesis blob and waypoint can be found at:
# `curl https://devnet.aptoslabs.com/waypoint.txt --output waypoint.txt`
# `curl https://devnet.aptoslabs.com/genesis.blob --output genesis.blob`
#
# Note this compose comes with a pre-configured node.config for fullnodes, see
# public_full_node.yaml. The config is pretty well documented and aligns with instructions herein.
# It is intended for use with testnet but can be easily modified for other systems.
#
# To start the docker compose:
# `docker-compose -f docker-compose.yaml up -d`
#
# Additional information:
# * If you use this compose for different Aptos Networks, you will need remove the db volume first.
# * Aptos's testnet produces approximately 3 GB worth of chain data per day, so be patient while
# starting for the first time. As a sanity check, enter the container and check the increasing size
# of the db:
#   * `docker exec -it $CONTAINER_ID /bin/bash`
#   * `du -csm /opt/aptos/data``
#
# Monitoring:
# If you want to install the monitoring components for your fullnode
# you can symlink the ../monitoring folder into this directory.
# Note that you will need to rename the monitoring docker-compose.yaml file to avoid duplication.
# e.g. rename it to docker-compose.mon.yaml and run:
# `docker-compose -f docker-compose.yaml -f docker-compose.mon.yaml up -d`
# Dashboard can be accessed locally by loading localhost:3000 on your browser
version: "3.8"
services:
  fullnode:
    image: aptoslab/validator:devnet
    volumes:
      - type: volume
        source: db
        target: /opt/aptos/data
      - type: bind
        source: ./genesis.blob
        target: /opt/aptos/etc/genesis.blob
        read_only: true
      - type: bind
        source: ./public_full_node.yaml
        target: /opt/aptos/etc/node.yaml
        read_only: true
      - type: bind
        source: ./waypoint.txt
        target: /opt/aptos/etc/waypoint.txt
        read_only: true
    command: ["/opt/aptos/bin/aptos-node", "-f", "/opt/aptos/etc/node.yaml"]
    ports:
      - "8080:8080"
      - "9101:9101"
      - "6180:6180"
volumes:
  db:

# Maintenant éditer le public-fullnode.yaml

base:
    # This is the location Aptos will store its database. It is backed by a dedicated docker volume
    # for persistence.
    data_dir: "/opt/aptos/data"
    role: "full_node"
    waypoint:
        # This is a checkpoint into the blockchain for added security.
        from_file: "/opt/aptos/etc/waypoint.txt"

state_sync:
  state_sync_driver:
    enable_state_sync_v2: true

execution:
    # Path to a genesis transaction. Note, this must be paired with a waypoint. If you update your
    # waypoint without a corresponding genesis, the file location should be an empty path.
    genesis_file_location: "/opt/aptos/etc/genesis.blob"

full_node_networks:
    - network_id: "public"
      discovery_method: "onchain"
      # The network must have a listen address to specify protocols. This runs it locally to
      # prevent remote, incoming connections.
      listen_address: "/ip4/0.0.0.0/tcp/6180"
      # Define the upstream peers to connect to
      seeds:
        {}

api:
    # This specifies your REST API endpoint. Intentionally on public so that Docker can export it.
    address: 0.0.0.0:8080
```

# Chargement des derniers fichiers genesis.blob et waypoint.txt :

```bash
wget https://devnet.aptolabs.com/genesis.blob
wget https://devnet.aptoslabs.com/waypoint.txt
```

# Démarrage du fullnode :

```bash
docker compose up -d
```



# PARTIE 2 : UPDATE DU FULLNODE A CHAQUE MISE A JOUR DU DEVNET

# En Général chaque vendredi une nouvelle version du devnet est déployée. Vous devez alors mettre à jour votre fullnode :

# C'est parti :

# Arreter votre FullNode :

```bash
docker compose stop
```

# Maintenant faire le ménage sur les images docker existantes :

```bash
docker compose down
```

# Lister les images docker existantes et notez les id :

```bash
docker images -a
```

# Supprimer les images :

```bash
docker image rm yourimageid -f
```

# Faire de meme avec les db volumes :
# Lister les volumes db existant et notez l'id :

```bash
docker volume ls
```

# Les supprimer :

```bash
docker volume rm volumeid -f
```

# Maintenant votre docker est "propre"

# Supprimer l'ancien genesis.blob et le waypoint.txt:

```bash
cd /root/aptos-fullnode
rm genesis.blob
rm waypoint.txt
```

# Downloader la dernière image docker mise à disposition : 

```bash
docker pull docker.io/aptoslabs/validator:devnet
```

# Maintenant downloader les nouveaux genesis.blob et waytpoint.txt :

```bash
wget https://devnet.aptoslabs.com/genesis.blob
wget https://devnet.aptoslabs.com/waypoint.txt
```

# Démarrer votre fullnode à jour :

```bash
docker compose up -d
```

Vous l'avez fait Bravo !!

# Pour vérifier que votre fullnode fonctionne bien :

# Se rendre sur :
 https://aptos-node.info et renseigner les champs avec votre adresse publique et le port 6180

# Rq: Si vous ne connaissez pas votre adresse publique tapez cette commande :

```bash
wget -qO- icanhazip.com
```

Pour toute question rendez-vous sur le TG Aptos France : https://t.me/AptosFrance
@kbconsulting45

A bientôt
