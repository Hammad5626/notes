# 🧑‍💻 Hamad Badar's Guide

## Table of Contents

1. 🔧 [Git and SSH](#git-and-ssh)
2. 🚀 [EC2 Deployment](#ec2-deployment)
3. 🐘 [Postgres Setup](#postgres-setup)
4. 🎨 [ZSH + OMZ + P10K](#zsh-setup)

---

## 🔧 GIT AND SSH <a id="git-and-ssh"></a>

### Git Config

```bash
git config --global user.name "root"
git config --global user.email "<root@example.com>"
```

### Generate SSH Key

```bash
ssh-keygen -t ed25519 -C "<root@example.com>"
```

### Start Agent and Add Key (LINUX)

```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
```

### Start Agent and Add Key (WINDOWS)

```bash
Get-Service -Name ssh-agent | Set-Service -StartupType Manual
Start-Service ssh-agent
ssh-add c:/Users/YOU/.ssh/id_ed25519
```

### Start Agent and Add Key (MAC)

```bash
eval "$(ssh-agent -s)"
open ~/.ssh/config
touch ~/.ssh/config
```

### UseKeychain yes only if you are using passphrase for keygen creation. Otherwise omit the line

```bash
Host github.com
  AddKeysToAgent yes
  UseKeychain yes
  IdentityFile ~/.ssh/id_ed25519
ssh-add --apple-use-keychain ~/.ssh/id_ed25519
```

### Add to github

### Linux

```bash
cat ~/.ssh/id_ed25519.pub
```

### Windows

```bash
clip < ~/.ssh/id_ed25519.pub
```

### Mac

```bash
pbcopy < ~/.ssh/id_ed25519.pub
```

---

## 🚀 EC2 Deployment <a id="ec2-deployment"></a>

### Required packages

```bash
sudo apt-get update
sudo apt-get install python3-pip python3-venv nginx supervisor -y
```

```python
python3 -m venv env
source env/bin/activate
```

### install gunicorn in your project envoirnment

```bash
pip3 install gunicorn
```

### install project req in the env

```bash
pip install django
```

### Supervisor configuration

```bash
sudo nano /etc/supervisor/conf.d/gunicorn.conf
```

```bash
[program:gunicorn]
directory=/home/ubuntu/pname
command=/home/ubuntu/pname/env/bin/gunicorn --workers 3 --bind unix:/home/ubuntu/pname/app.sock --timeout 600 core.wsgi:application
autostart=true
autorestart=true
stderr_logfile=/logs/gunicorn.err.log
stdout_logfile=/logs/gunicorn.out.log
```

```bash
sudo mkdir /logs
```

### Verify Supervisor configuration

```bash
sudo supervisorctl reread
sudo supervisorctl update
sudo supervisorctl status
```

### NGINX configuration changes

```bash
sudo nano /etc/nginx/sites-available/django.conf
```

```bash
server{
 listen 80;
 server_name addr;
 location / {
  include proxy_params;
  proxy_read_timeout 300s;
  proxy_connect_timeout 120s;
  proxy_pass <http://unix:/home/ubuntu/pname/app.sock>;
 }
 location /static/ {
  autoindex on; 
  alias /home/ubuntu/pname/static/;
 }
 location /media/ {
  autoindex on; 
  alias /home/ubuntu/pname/media/;
 }
}
```

### Verify NGINX config changes and restart server

```bash
sudo nano /etc/nginx/nginx.conf
```

```bash
user ubuntu;
increase server_names_hash_bucket_size: 128;
client_max_body_size 200M;
sudo nginx -t
sudo ln /etc/nginx/sites-available/django.conf /etc/nginx/sites-enabled
sudo service nginx restart
```

### Update Server

```bash
sudo supervisorctl reload
sudo service nginx restart
```

### Checking error log or output log

```bash
tail -n 50 /logs/gunicorn.err.log
tail -n 50 /logs/gunicorn.out.log
```

### Certbot

```bash
sudo apt-get install snapd
sudo snap install --classic certbot
sudo ln -s /snap/bin/certbot /usr/bin/certbot
sudo certbot --nginx
```

---

## 🐘 Postgres Setup <a id="postgres-setup"></a>

### ArchLinux

```bash
sudo pacman -S postgresql
sudo -iu postgres initdb --locale en_US.UTF-8 -D /var/lib/postgres/data
```

### Deb or Ubuntu

```bash
sudo apt update
sudo apt install postgresql postgresql-contrib
```

### Enable and Start PostgreSQL

```bash
sudo systemctl enable postgresql
sudo systemctl start postgresql
```

### Create User and DB

```bash
sudo -iu postgres
psql
CREATE USER sampleuser WITH PASSWORD 'samplepasswd';
ALTER USER sampleuser WITH SUPERUSER;
CREATE DATABASE sampledb;
GRANT ALL PRIVILEGES ON DATABASE sampledb TO sampleuser;
\q
```

---

## 🎨 ZSH + OMZ + P10K <a id="zsh-setup"></a>

### install zsh

```bash
sudo apt-get install zsh
sudo pacman -S zsh
```

### Change Default Shell to ZSH

```bash
chsh -s $(which zsh)
```

### Start ZSH

```bash
zsh
```

### Install OMZ

```bash
sh -c "$(curl -fsSL <https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh>)"
```

### Install P10K

```bash
git clone <https://github.com/romkatv/powerlevel10k.git> $ZSH_CUSTOM/themes/powerlevel10k
```

### Change theme to P10K

```bash
ZSH_THEME="powerlevel10k/powerlevel10k"
```

### Install FiraCode or DroidSansM from NerdFonts

[DroidSansM](https://github.com/ryanoasis/nerd-fonts/releases/download/v3.2.1/DroidSansMono.zip)
[FiraCode](https://github.com/ryanoasis/nerd-fonts/releases/download/v3.2.1/FiraCode.zip)

or download using wget

```bash
wget https://github.com/ryanoasis/nerd-fonts/releases/download/v3.2.1/DroidSansMono.zip
wget https://github.com/ryanoasis/nerd-fonts/releases/download/v3.2.1/FiraCode.zip
```

```bash
unzip DroidSansM.zip
cd DroidSansM
sudo mkdir -p /usr/share/fonts/truetype/droidsans
sudo mv DroidSansM.ttf /usr/share/fonts/truetype/droidsans/
sudo fc-cache -fv
```

Using the font in Gnome Terminal

```bash
gsettings set org.gnome.desktop.interface monospace-font-name "DroidSansM 12"
```

### Configure P10K

```bash
p10k configure
```

### Install SyntaxHighlighter and AutoSuggestions

```bash
git clone <https://github.com/zsh-users/zsh-syntax-highlighting.git> ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
```

### Install ColorLS and EXA

For ArchLinux

```bash
sudo pacman -S ruby
sudo pacman -S exa
sudo gem install colorls
```

For Ubuntu or Debian Based Linux

```bash
sudo apt install ruby-full
sudo gem install colorls
sudo apt install -y cargo
cargo install exa
```

### Pluggins for ZSH

```bash
plugins=( git zsh-syntax-highlighting zsh-autosuggestions  z python archlinux gitignore pip aliases aliases catimg colorize git-auto-fetch git-prompt zsh-interactive-cd vscode virtualenvwrapper virtualenv urltools universalarchive themes supervisor supervisor sudo snap singlechar sfffe safe-paste rand-quote qrcode lol emoji-clock emoji)
```

### Aliasing

```bash
alias ls='exa --icons'
alias dir='exa --icons'
alias ..='cd ..'
```

### Complete Aliasing for me

```bash
alias ls='exa --icons'
alias dir='exa --icons'
alias coc='waydroid app launch com.supercell.clashofclans'
alias waydroid-reload='sudo systemctl restart waydroid-container.service'
alias fpi='flatpak install'
alias fpr='flatpak remove'
alias fpu='flatpak update'
alias pmr='python manage.py runserver'
alias pmr9='python manage.py runserver 9000'
alias pmr8='python manage.py runserver 8080'
alias pmk='python manage.py makemigrations'
alias pmg='python manage.py migrate'
alias pmsu='python manage.py createsuperuser'
alias mkenv='python -m venv env'
alias env='source ./env/bin/activate'
alias denv='deactivate'
alias zshrc='nano ~/.zshrc'
alias czshrc='code ~/.zshrc'
alias szsh='source ~/.zshrc'
alias msinfo32='neofetch'
alias taskmgr='htop'
alias ..='cd ..'
alias sshfam='sudo ssh -i ~/Documents/pem/FamilyArchive.pem ubuntu@13.48.43.200'
alias sshsteno='sudo ssh -i ~/Documents/pem/steno_prep.pem ubuntu@54.151.46.192'
alias sshprop='sudo ssh -i ~/Documents/pem/propgul.pem ubuntu@18.117.244.62'
alias sshmqtt='sudo ssh -i ~/Documents/pem/mqtt.pem ubuntu@13.48.185.60'
```

### Package Manager Aliasing

```bash
function apt() {
    if [[ $1 == "install" ]]; then
        shift
        sudo pacman -S "$@"
    elif [[ $1 == "remove" ]]; then
        shift
        sudo pacman -R "$@"
    elif [[ $1 == "update" ]]; then
        shift
        sudo pacman -Sy "$@"
    elif [[ $1 == "upgrade" ]]; then
        shift
        sudo pacman -Syu "$@"
    else
        echo "Command not supported: dnf $@"
    fi
}
```

```bash
function dnf() {
    if [[ $1 == "install" ]]; then
        shift
        sudo pacman -S "$@"
    elif [[ $1 == "remove" ]]; then
        shift
        sudo pacman -R "$@"
    elif [[ $1 == "update" ]]; then
        shift
        sudo pacman -Sy "$@"
    elif [[ $1 == "upgrade" ]]; then
        shift
        sudo pacman -Syu "$@"
    else
        echo "Command not supported: dnf $@"
    fi
}
```

```bash
function app() {
    if [[ $1 == "i" ]]; then
        shift
        sudo pamac install "$@"
    elif [[ $1 == "r" ]]; then
        shift
        sudo pamac remove "$@"
    elif [[ $1 == "up" ]]; then
        shift
        sudo pamac update "$@"
    elif [[ $1 == "u" ]]; then
        shift
        sudo pamac upgrade "$@"
    else
        echo "Command not supported: dnf $@"
    fi
}
```

```bash
function sudo() {
    if [[ $1 == "apt" ]]; then
        shift
        apt "$@"
    elif [[ $1 == "dnf" ]]; then
        shift
        dnf "$@"
    else
        command sudo "$@"
    fi
}
```

```bash
function sudo() {
    if [[ $1 == "dnf" ]]; then
        shift
        dnf "$@"
    else
        command sudo "$@"
    fi
}
```
