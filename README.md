<h1 align="center">Incentivized Node: Ora Node</h1>

<p align="center">
  <img src="https://github.com/user-attachments/assets/a5069cfd-6dc6-4919-8bb3-d71aecebba39" alt="Ora Protocol">
</p>

**Ora Protocol** is an AI Oracle computing data for blockchain smart contracts using decentralized AI models.

- 💰 **Funding**: Raised **$20 million** from top investors like **Polychain** and **Hashkey**.
- 🤝 **Partnerships**: Collaborating with major blockchain platforms such as **Optimism**, **Arbitrum**, and **Base**.


# Install Tora Node

## Hardware Requirements
| Ram | cpu     | disk                      |
| :-------- | :------- | :-------------------------------- |
| `min. 8 GB`      | `1 Core` | `20-40 GB SSD` |

## Dependecies
Install Docker
```console
sudo apt update -y && sudo apt upgrade -y
for pkg in docker.io docker-doc docker-compose podman-docker containerd runc; do sudo apt-get remove $pkg; done

sudo apt-get update
sudo apt-get install ca-certificates curl gnupg
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update -y && sudo apt upgrade -y

sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Docker version check
docker --version
```

## Obtain Alchemy API
* Create account here: https://dashboard.alchemy.com/
* Create and `Ethereum` API and copy `HTTPS` and `Websockets` for `Sepolia` & `Mainnet` 

![Screenshot_8](https://github.com/user-attachments/assets/7912f1f1-4884-4d03-b263-e96a0c58043e)

## Install Tora
### Create Tora directory
```console
mkdir tora
cd tora
```

### Create .env file
```console
nano .env
```

**Paste the following code in .env**
* `PRIV_KEY`: Replace a Metamask wallet private key
* Ensure you have some ETH on Sepolia & Mainnet to pay 1 transaction gas 
* `MAINNET_WSS`, `MAINNET_HTTP`,`SEPOLIA_WSS`, `SEPOLIA_HTTP`: Paste your `Alchemy` URLs
```
############### Sensitive config ###############

# private key for sending out app-specific transactions
PRIV_KEY=""

############### General config ###############

# general - execution environment
TORA_ENV=production

# general - provider url
MAINNET_WSS=""
MAINNET_HTTP=""
SEPOLIA_WSS=""
SEPOLIA_HTTP=""

# redis global ttl, comment out -> no ttl limit
REDIS_TTL=86400000 # 1 day in ms 

############### App specific config ###############

# confirm - general
CONFIRM_CHAINS='["sepolia"]' # sepolia | mainnet ｜ '["sepolia","mainnet"]'
CONFIRM_MODELS='[13]' # 13: OpenLM ,now only 13 supported
# confirm - crosscheck
CONFIRM_USE_CROSSCHECK=true
CONFIRM_CC_POLLING_INTERVAL=3000 # 3 sec in ms
CONFIRM_CC_BATCH_BLOCKS_COUNT=300 # default 300 means blocks in 1 hours on eth
# confirm - store ttl
CONFIRM_TASK_TTL=2592000000
CONFIRM_TASK_DONE_TTL = 2592000000 # comment out -> no ttl limit
CONFIRM_CC_TTL=2592000000 # 1 month in ms
```
* `CTRL+X+Y+ENTER` to save and exit

### Create docker-compose file
```console
nano docker-compose.yml
```

**Paste following code**
```
# ora node docker-compose
version: '3'
services:
  confirm:
    image: oraprotocol/tora:confirm
    container_name: ora-tora
    depends_on:
      - redis
      - openlm
    command: 
      - "--confirm"
    env_file:
      - .env
    environment:
      REDIS_HOST: 'redis'
      REDIS_PORT: 6379
      CONFIRM_MODEL_SERVER_13: 'http://openlm:5000/'
    networks:
      - private_network
  redis:
    image: oraprotocol/redis:latest
    container_name: ora-redis
    restart: always
    networks:
      - private_network
  openlm:
    image: oraprotocol/openlm:latest
    container_name: ora-openlm
    restart: always
    networks:
      - private_network
  diun:
    image: crazymax/diun:latest
    container_name: diun
    command: serve
    volumes:
      - "./data:/data"
      - "/var/run/docker.sock:/var/run/docker.sock"
    environment:
      - "TZ=Asia/Shanghai"
      - "LOG_LEVEL=info"
      - "LOG_JSON=false"
      - "DIUN_WATCH_WORKERS=5"
      - "DIUN_WATCH_JITTER=30"
      - "DIUN_WATCH_SCHEDULE=0 0 * * *"
      - "DIUN_PROVIDERS_DOCKER=true"
      - "DIUN_PROVIDERS_DOCKER_WATCHBYDEFAULT=true"
    restart: always

networks:
  private_network:
    driver: bridge
```
* `CTRL+X+Y+ENTER` to save and exit

## Run Tora Node
```console
docker compose up -d
```

## Check logs
```console
cd $HOME && cd tora
docker compose logs -f -n 100
```

Check Tora logs
```console
docker logs -f -n 100 ora-tora
```
