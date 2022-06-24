# Aptos_Validator and Fullnode
**# The guide is made based on document of Aptos team. I just customize clearly for setting up fullnode and validator node on same machine**
**# The guide is used for Docker only**

**Step 1: Install Docker and Docker-Compose, Aptos CLI.**

    apt update && apt install git sudo unzip wget -y
    
    ### install docker
      curl -fsSL https://get.docker.com -o get-docker.sh
      sudo sh get-docker.sh
      
    ### install docker-compose
      curl -SL https://github.com/docker/compose/releases/download/v2.5.0/docker-compose-linux-x86_64 -o /usr/local/bin/docker-compose
      sudo chmod +x /usr/local/bin/docker-compose
      sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose

    ### install aptos
      wget -qO aptos-cli.zip https://github.com/aptos-labs/aptos-core/releases/download/aptos-cli-v0.1.1/aptos-cli-0.1.1-Ubuntu-x86_64.zip
      unzip -o aptos-cli.zip
      chmod +x aptos
      mv aptos /usr/local/bin

**Step 2: Create a directory for your Aptos node composition**

    export WORKSPACE=validator
    mkdir ~/$WORKSPACE
    cd ~/$WORKSPACE
    
 **Step 3: Download the validator.yaml and docker-compose.yaml configuration files into this directory**
 
    wget https://raw.githubusercontent.com/aptos-labs/aptos-core/main/docker/compose/aptos-node/docker-compose.yaml
    wget https://raw.githubusercontent.com/aptos-labs/aptos-core/main/docker/compose/aptos-node/validator.yaml
 
 **Step 4: Generate key pairs (node owner key, consensus key and networking key) in your working directory.**
 
    aptos genesis generate-keys --output-dir ~/$WORKSPACE
    
 **Step 5: Configure validator information**
 
    export IPADDR=`curl ifconfig.me`
    export VALIDATOR_NAME=<YOUR_NAME>
    aptos genesis set-validator-configuration \
    --keys-dir ~/$WORKSPACE --local-repository-dir ~/$WORKSPACE \
    --username $VALIDATOR_NAME \
    --validator-host $IPADDR:6180 \
    --full-node-host $IPADDR:6182
  
  **Step 6: Create layout YAML file, which defines the node in the validatorSet, for test mode, we can create a genesis blob containing only one node**
  
    echo "---
    root_key: \"0x5243ca72b0766d9e9cbf2debf6153443b01a1e0e6d086c7ea206eaf6f8043956\"
    users:
      - $VALIDATOR_NAME
    chain_id: 23" > ~/$WORKSPACE/layout.yaml
    
 **Step 7: Download AptosFramework Move bytecodes.**
 
    cd ~/$WORKSPACE && wget https://github.com/aptos-labs/aptos-core/releases/download/aptos-framework-v0.1.0/framework.zip
    unzip framework.zip

**Step 8: Compile genesis blob and waypoint**

    aptos genesis generate-genesis --local-repository-dir ~/$WORKSPACE --output-dir ~/$WORKSPACE
    
**Step 9: Check whether the below files are in your working directory**
    
    ls -l
    total 116
    -rw-r--r-- 1 root root  1185 May 17 09:17 docker-compose.yaml
    drwxr-xr-x 2 root root  4096 May 12 15:23 framework
    -rw-r--r-- 1 root root 30588 May 12 22:50 framework.zip
    -rw-r--r-- 1 root root   556 May 17 09:19 gauthuikute.yaml
    -rw-r--r-- 1 root root 46993 May 17 09:21 genesis.blob
    -rw-r--r-- 1 root root   119 May 17 09:20 layout.yaml
    -rw-r--r-- 1 root root   436 May 17 09:18 private-keys.yaml
    -rw-r--r-- 1 root root   168 May 17 09:18 validator-full-node-identity.yaml
    -rw-r--r-- 1 root root   334 May 17 09:18 validator-identity.yaml
    -rw-r--r-- 1 root root  1074 May 17 09:17 validator.yaml
    -rw-r--r-- 1 root root    66 May 17 09:21 waypoint.txt
    
**Step 10: Run validator node**

    docker compose up -d

**Step 11: Setup Fullnode on a same machine of Validator node, then Download the fullnode.yaml and docker-compose-fullnode.yaml**

    export WORKSPACE_FULLNODE=fullnode
    mkdir ~/$WORKSPACE_FULLNODE
    cd ~/$WORKSPACE_FULLNODE
    wget -qO docker-compose.yaml https://raw.githubusercontent.com/aptos-labs/aptos-core/main/docker/compose/aptos-node/docker-compose-fullnode.yaml
    wget https://raw.githubusercontent.com/aptos-labs/aptos-core/main/docker/compose/aptos-node/fullnode.yaml
    cd ~/$WORKSPACE && cp -rf validator-full-node-identity.yaml genesis.blob waypoint.txt ~/$WORKSPACE_FULLNODE && cd -
    
**Step 12: Edit some parameter in fullnode related files**
    
    export IPADDR=`curl ifconfig.me`
    sed -i '3,3 s|\(/opt/aptos/data\)|\1_fullnode|g' fullnode.yaml
    sed -i 's|<Validator IP Address>|'$IPADDR'|g' fullnode.yaml
    sed -i 's|9101|9103|' docker-compose.yaml
    sed -i 's|\(/opt/aptos/data\)|\1_fullnode|' docker-compose.yaml
    
**Step 13: Run Fullnode**
    
    cd ~/$WORKSPACE_FULLNODE && docker compose up -d
    
**Step 14: Check status of Fullnode and Validator node**

    ### Check LISTEN PORT 6180, 6182, 9101, 9103
    sudo lsof -i -P -n | grep LISTEN | grep "6180\|6182\|9101\|9103\|6181"
    
    ### Check state sync 
    curl 127.0.0.1:9101/metrics 2> /dev/null | grep aptos_state_sync_version
    curl 127.0.0.1:9103/metrics 2> /dev/null | grep aptos_state_sync_version
    
    ### Login web https://aptos-node.info/ then check. Fill port 6180/9101 for VALIDATOR, 6182/9103 for FULLNODE
    
    
