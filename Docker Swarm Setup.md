# Set up Docker Swarm

> Node password - *CWX6bKveoEF^$

> PiHole password - washing-ignition-implosion

> PiHole Adlist - https://github.com/hagezi/dns-blocklists?tab=readme-ov-file#pro

> Docker Relaod script after shutdown - ```./docker_loader.sh ```

### 1. Set static IP on the clsuter network

```bash
$ sudo nmcli con mod "Wired connection 1" ipv4.addresses 192.168.50.1/24 ipv4.method manual     MASTER
$ sudo nmcli con mod "Wired connection 1" ipv4.addresses 192.168.50.2/24 ipv4.method manual     WORKER1
$ sudo nmcli con mod "Wired connection 1" ipv4.addresses 192.168.50.3/24 ipv4.method manual     WORKER2
$ sudo nmcli con down "Wired connection 1"
$ sudo nmcli con up "Wired connection 1"
```
### 2. Check eth0 with nmcli
```bash
$ nmcli
```

### 3) Set up shutdown script

3.1) Generate ssh keys
```bash
$ ssh-keygen
```
3.2) Copy public key to workers
```bash
$ ssh-copy-id pi@192.168.50.2
$ ssh-copy-id pi@192.168.50.3
```
3.3) Test by ssh into workers (should not need a password)
```bash
$ ssh pi@192.168.50.2
$ ssh pi@192.168.50.2
```
3.4) Create file
```bash
$ nano shutdown_nodes.sh
```
3.5) Add the script and save
```bash
#!/bin/bash

#worker IPs
workers=("192.168.50.2" "192.168.50.3")

#ssh into the workers and shut down
for worker in "${workers[@]}"; do
    ssh pi@$worker "sudo shutdown -h now"
done
```
3.6) Make the script executeable
```bash
$ chmod +x shutdown_nodes.sh
```
3.7) Test the script
```bash
./shutdown_nodes.sh
```

### 4) Install Docker on each Node

4.1) Set up Docker apt repo
```bash
$ sudo apt-get update
$ sudo apt-get install ca-certificates curl
$ sudo install -m 0755 -d /etc/apt/keyrings
$ sudo curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
$ sudo chmod a+r /etc/apt/keyrings/docker.asc
```
4.2) Add the repository to Apt sources
```bash
echo \
"deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/debian \
$(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

$ sudo apt-get update
```
4.2) Install Docker packages
```bash
$ sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```
4.3) Verify installation with hello-world image
```bash
$ sudo docker run hello-world
```
### 5) Set up the Swarm

5.1) Initialise Docker Swarm on master node
```bash
$ sudo docker swarm init --advertise-addr 192.168.50.1
```
5.2) Use the worker node join command generated by the initialise command (may have to use sudo on the workers)
```bash
$ sudo docker swarm join --token <TOKEN> <MASTER_NODE_IP>:2377
```
5.3) Check Swarm status
```bash
sudo docker node ls
```
### 6) Instll Portainer on master node

6.1) Create Portainer yaml config file
```bash
$ nano portainer.yaml
```
6.2) Add the following to the file (if this doesnt work use chatgpt version - https://chat.openai.com/share/40e8966a-371d-417b-bf09-acc7ef756bfe)
```yaml
version: '3.8'

services:
portainer:
    image: portainer/portainer-ce:latest
    deploy:
    mode: replicated
    replicas: 1
    placement:
        constraints:
        - node.role == manager
    ports:
    - "9000:9000"
    volumes:
    - "/var/run/docker.sock:/var/run/docker.sock"
    - "portainer_data:/data"

volumes:
portainer_data:
```
    
6.3) Deploy the Portainer stack
```bash
$ sudo docker stack deploy -c docker-compose.yml portainer
```
6.4) Log into Portainer web interface 

> 192.168.1.150:9000
