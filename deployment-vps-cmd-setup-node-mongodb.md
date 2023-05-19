# Working with nodejs-mongodb Project with Virtual Private Server (linux VPS) in Terminal

### access vps using ssh

```sh

ssh <username>@<ip address|domainname>

```

### install nodejs and npm

```sh
sudo apt install -y nodejs npm
```

### install mongodb server

```sh
wget -qO - https://www.mongodb.org/static/pgp/server-6.0.asc | sudo apt-key add -
sudo apt-get install gnupg
wget -qO - https://www.mongodb.org/static/pgp/server-6.0.asc | sudo apt-key add -
echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu focal/mongodb-org/6.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-6.0.list
sudo apt-get update
sudo apt-get install -y mongodb-org
```

### install libssl dependency of mongodb

```sh
echo "deb http://security.ubuntu.com/ubuntu focal-security main" | sudo tee /etc/apt/sources.list.d/focal-security.list
sudo apt-get update
sudo apt-get install libssl1.1
sudo rm /etc/apt/sources.list.d/focal-security.list
sudo apt-get install -y mongodb-org

```

### install npm forever

```sh
sudo npm i -g forever
```

### allowing important port in vps linux firewall

#### if ufw is not installed

```sh
sudo apt install ufw
```

##### note: know the important port that used connecting with your vps such as ssh port, you need to set it

```sh
sudo ufw allow ssh
#note: this port use in accessing vps dashboard in godaddy
sudo ufw allow 2224/tcp

# allow mongodb port in ufw

sudo ufw allow 27017/tcp

# allow http or https port

sudo ufw allow https
sudo ufw allow http
sudo ufw enable
sudo ufw reload
```

##### note: if the port you configure not working, restart the vps server in you dashboard

## reconnect with the vps using ssh

### run mongodb as services

```sh
sudo service mongod start
```

## set storage cache_size

```yml
wiredTiger:
engineConfig:
configString: cache_size=600M
```

### run nodejs as a foreground using forever

```sh
sudo forever start app.js
```

### update nodejs repo

```sh
cd <project>
git pull origin main
sudo forever restart app.js
```
