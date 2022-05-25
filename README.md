# cosmozab
Zabbix stuff for Cosmos Nodes

WARNiNG: This is still very much in development and i have not yet created most of the documentation to get up and running. Assuming you are proficient in Zabbix somewhat, you should be able to get these working. You will need to add macros etc for some of the functionality. Just ask if there is any questions.

I have developed using Zabbix 6.0 server/gui and zabbix-agen2. But should work on any version above 5 i think.

Requires:
- API and RPC exposed for localhost. Calls will be made to these services from zabbix-agent2. Refer to the `zab.userparameters.cosmos.conf` file for info on what calls are made to the API and RPC.

## Introduction
This is a collection of instructions, configurations and templates to integrate Zabbix monitoring with standard Cosmos Nodes running.

Configurations are based on Ubuntu 20.04 but should work on most Linux distributions.

## Screenshots
![image](https://user-images.githubusercontent.com/85548668/147846304-386e1123-eab5-4eea-89dd-fb5d85fb352d.png)


# Install Agent and Configs
We will be using Zabbix version 5.4 repository.

## Install repository and zabbix-agent2
Add repository and update `apt`:
```bash
wget https://repo.zabbix.com/zabbix/5.4/ubuntu/pool/main/z/zabbix-release/zabbix-release_5.4-1+ubuntu20.04_all.deb
sudo dpkg -i zabbix-release_5.4-1+ubuntu20.04_all.deb && sudo apt update
rm -rf zabbix-release_5.4-1+ubuntu20.04_all.deb
```
Install zabbix-agent2 with `apt`:
```bash
sudo apt install zabbix-agent2 -y
```

## Configure zabbix-agent2
Edit the zabbix-agen2.conf

Set a variable with your server address:
```bash
SERVER=<your-server-address>
```

Inject server address into config with `sed`
```bash
sudo sed -i 's/Server=127.0.0.1/Server='"${SERVER}"'/g' /etc/zabbix/zabbix_agent2.conf
```

## Install Configs

`curl` config file from repo

```bash
sudo curl -o /etc/zabbix/zabbix_agent2.d/zab.userparameters.cosmos.conf https://raw.githubusercontent.com/gh0stdotexe/cosmozab/main/zabbix_agent2.d/zab.userparameters.cosmos.conf
```

Update UFW for Zabbix port
```bash
sudo ufw allow from <zabbix-server-address> to any port 10050
```

Enable and restart agent
```bash
sudo systemctl daemon-reload && sudo systemctl enable zabbix-agent2
sudo systemctl restart zabbix-agent2
journalctl -u zabbix-agent2 -f
```

Clean Up Existing Monitoring (Prometheus, Node Exporter)

Disable old services:
```bash
sudo systemctl disable node_exporter
sudo systemctl disable prometheus
```

Close port 9093 (old Prometheus port):
```bash
sudo ufw status numbered

# delete both 9093 ports from list (example #'s below)

sudo ufw delete #4
sudo ufw delete #9
```
