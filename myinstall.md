# How I set my system from zero
after installing Ubuntu 16.04 I installed `vim`, `atom` and `GDebi`. `Gdebi` is a software for installation `deb` packages.
## NVidia settings
For getting list of proposed versions of driver:
```
ubuntu-drivers devices
```
Usually better install _recommended_ drivers. To install it we need:
```
sudo apt-get update && sudo apt-get install nvidia-123
```
where **123 is version** of selected driver

After driver loaded needs to reload pc:
```
sudo reboot
```
After rebooting press `win/super` or search in program menu our nvidia. And inside nvidia program search current version of driver. If version is the same as we try to install - driver was installed correctly. If not - we can try to install version from nvidia website.

### Fix the apt-get install error: “Media change: please insert the disc labeled...
If we got such error we need to fix our boot. Open our `/etc/apt/sources.list` file. I'll use `vim` you can use `gedit` if you want.
```
sudo vim /etc/apt/sources.list
```
or
```
sudo gedit /etc/apt/sources.list
```
Look on line 3. He say that need to use cdrom. Comment it by adding sharp(`#`) in start of the line.
```
# deb cdrom:[Ubuntu 16.04.4 LTS _Xenial Xerus_ - Release amd64 (20180228)]/ xenial main restricted
```
## Web browsers
### Opera
Full installation guide [was here](http://arc.opera.com/manual.html).
```
sudo add-apt-repository 'deb https://deb.opera.com/opera-stable/ stable non-free'
wget -qO- https://deb.opera.com/archive.key | sudo apt-key add -
```
and after
```
sudo apt-get update
sudo apt-get install opera-stable
```
### Chromium
We must install it. But it easy can be done by application manager.
### Flashplayer
> **Close all browsers!!!**

Next:
```
sudo apt update
sudo apt install adobe-flashplugin
```
## Communications
### Skype
I think now or later you'll need **curl** so:
```
sudo apt-get install curl
```
And next add skype:
```
curl https://repo.skype.com/data/SKYPE-GPG-KEY | sudo apt-key add -
echo "deb https://repo.skype.com/deb stable main" | sudo tee /etc/apt/sources.list.d/skypeforlinux.list
```
and we can install
```
sudo apt update
sudo apt install skypeforlinux
```
### telegram
```
sudo add-apt-repository ppa:atareao/telegram
sudo apt-get update
sudo apt-get install telegram
```
Telegram couldn't open desktop version, instead it launch the web version. So we must run following command :
```
/opt/telegram/Telegram
```
### Slack
Install from application manager.
## GitHub
I njoy to work with github, so, first of all, I'll install it.

Install git:
```
sudo add-apt-repository ppa:git-core/ppa
sudo apt update; sudo apt install git
```
Setup git
```
sudo touch ~/.gitconfig
sudo vim ~/.gitconfig
```
Base config:
```
[user]
	name = Mikita Melnikau
	email = usesa@yandex.com
	username = pinkyrabbit
[core]
	editor = vim
	whitespace = fix,-indent-with-non-tab,trailing-space,cr-at-eol
	excludesfile = ~/.gitignore
[sendemail]
	smtpencryption = tls
	smtpserver = smtp.gmail.com
	smtpuser = usesa@yandex.com
	smtppass = password
	smtpserverport = 587
[web]
	browser = chromium-browser
[rerere]
	enabled = 1
	autoupdate = 1
[push]
	default = matching
[color]
	ui = auto
[color "branch"]
	current = yellow bold
	local = green bold
	remote = cyan bold
[color "diff"]
	meta = yellow bold
	frag = magenta bold
	old = red bold
	new = green bold
	whitespace = red reverse
[color "status"]
	added = green bold
	changed = yellow bold
	untracked = red bold
[github]
	user = pinkyrabbit
	token = token
```
Dont forget let token to [control repo](https://help.github.com/articles/creating-a-personal-access-token-for-the-command-line/)

Next I need GitFlow:
```
apt-get install git-flow
```
Here is [link to repo](https://github.com/nvie/gitflow). Reopen terminal.
And **Git Kraken**:
```
wget https://release.gitkraken.com/linux/gitkraken-amd64.deb
sudo dpkg -i gitkraken-amd64.deb
sudo apt-get install -f
rm -f gitkraken-amd64.deb
```
## Working flow
### NodeJS & NVM
Node:
```
curl -sL https://deb.nodesource.com/setup_8.x | sudo -E bash -
sudo apt-get install -y nodejs
```
We need `build-essential` now or later so install it (if still not):
```
sudo apt-get install -y build-essential
```
GitHub [NVM page](https://github.com/creationix/nvm).
```
sudo touch ~/.bash_profile
sudo wget -qO- https://raw.githubusercontent.com/creationix/nvm/v0.33.8/install.sh | bash
```
Reload terminal for apply `nvm`.
> How to set npm permissions guide [here ->](https://docs.npmjs.com/getting-started/fixing-npm-permissions)

### misc
```
npm install -g eslint
sudo apt-get install cmake
sudo add-apt-repository ppa:jonathonf/gcc-7.1
sudo apt-get update
sudo apt-get install gcc-7 g++-7
sudo apt-get install python-dev libxml2-dev libxslt-dev
```

### yarn
```
curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -
echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list
sudo apt update
sudo apt install yarn
```

### Vim
My Vim config [is here](https://github.com/PinkyRabbit/javascript-node-react-vim-config)
### Atom
Download deb from website and install with `gdebi`.
* `atom-material-ui` + `atom-material-syntax` - that what I use. Add contrastig and shadows to my panels insettings, and maked tabs compact.
* `file-icons` for better lookup.
* `linter-eslint` for eslint

## Bonus
### Filezilla
```
sudo wget -q -O - http://archive.getdeb.net/getdeb-archive.key | sudo apt-key add
sudo apt update
sudo apt install filezilla
```
### Remmina
Has to be installed by deafult

## Themes
We need `unity-tweak-tool`
```
sudo apt-get install unity-tweak-tool
```
Next we install theme:
```
sudo add-apt-repository ppa:dyatlov-igor/materia-theme
sudo apt update
sudo apt install materia-gtk-theme
```
And make settings into Unity Tweak Tool. I joined:
* Theme - Materia
* Icons - Ubuntu-mono-dark
* Terminal colors - white on black
