## Basic Overview
* restart_preiskampf_ui_service.sh sits in /opt/scripts
* preiskampf-app.service sits in /etc/systemd/system and is added via systemctl
* DNS Config is in Cloudflare Dashboard
* Caddy Config sits in /etc/caddy/Caddyfile
* github actions need server ip in secrets for deployment


## Manual Steps, Preparation
* do ssh-keygen and add key to github via dashboard
* use root to create a user "preiskampf"
```
adduser preiskampf
usermod -aG sudo preiskampf
rsync --archive --chown=preiskampf:preiskampf ~/.ssh /home/preiskampf
```
* checkout preiskampf-config and preiskampf-app
```
mkdir projects
cd projects
git clone git@github.com:VaJulianLindner/preiskampf-config.git
git clone git@github.com:VaJulianLindner/preiskampf-app.git
```
* create DNS Records for the static ip
* install pkgs to compile openssl into binary
```
sudo apt install openssl libssl-dev pkg-config
```


## Script Steps (to be implemented)
* install caddy
```
sudo apt install -y debian-keyring debian-archive-keyring apt-transport-https curl
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/gpg.key' | sudo gpg --dearmor -o /usr/share/keyrings/caddy-stable-archive-keyring.gpg
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/debian.deb.txt' | sudo tee /etc/apt/sources.list.d/caddy-stable.list
sudo apt update
sudo apt install caddy
```
* install nvm
```
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.1/install.sh | bash
source ~/.bashrc
nvm install v20.11.1
```
* install rust
```
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
sudo apt install build-essential
```
* install/setup script to run everything, distribute this repo's files to destination dirs, add env DB_USER and DB_PASSWORD in service config file
```
cp ~/projects/preiskampf-config/Caddyfile /etc/caddy/Caddyfile
sudo systemctl reload caddy
systemctl status caddy
```s/restart_preiskampf_ui_service.sh
```
```
sudo cp ~/projects/preiskampf-config/preiskampf-app.service /etc/systemd/system/preiskampf-app.service
sudo systemctl enable preiskampf-app
sudo systemctl daemon-reload
sudo systemctl start preiskampf-app
```
* restarting the service is a change to systemd and thus requires sudo. Setup passwordless sudo for preiskampf user then copy the script to it's location
```
sudo visudo
>> preiskampf ALL=(ALL) NOPASSWD: ALL
```
```
sudo mkdir /opt/scripts
sudo cp ~/projects/preiskampf-config/restart_preiskampf_ui_service.sh /opt/script
```