# Custom Ubuntu Distro for WSL

This is a quick repo for me to build out my dev environment in WSL. Planning to create scripts based on the instructions in here.

Adapted from [https://learn.microsoft.com/en-us/windows/wsl/use-custom-distro](https://learn.microsoft.com/en-us/windows/wsl/use-custom-distro)

## Requirements

- Docker for Windows - [https://docker.com/download](https://docker.com/download)
- PowerShell Core - [https://github.com/PowerShell/PowerShell](https://github.com/PowerShell/PowerShell)
- Windows terminal - [https://github.com/microsoft/terminal](https://github.com/microsoft/terminal)

## 1. Create tar file

For the latest LTS version (22.04). All commands to be run in powershell.

```pwsh
docker run -t ubuntu:22.04 /usr/bin/ls
```

Get the container ID.

```pwsh
docker container ls -a
```

Export to a tar file.

```pwsh
docker export [id] > C:\path\to\ubuntu.tar
```

Where `[id]` is the container ID.

## 2. Import to WSL

```pwsh
wsl --import DevMachine C:\path\to\install\dir C:\path\to\ubuntu.tar
```

Re-open windows terminal and check if you can enter the container.

```pwsh
wsl -d DevMachine
```

If you enter into the root shell then it's successfully installed.

## 3. Container setup

> [!IMPORTANT]
> We are now inside the ubuntu container. So powershell commands will not work in the container.

Setup the container for the user, while inside the container. Uncompress the container since it was
a docker container before, we want the full ubuntu experience now.

```sh
unminimize
```

Install core tools for the devlopment environment.

```sh
apt install -y nano vim curl wget unzip zip git man-db locales locate build-essential autoconf sudo systemd systemd-sysv iproute2 python3 python3-pip
```

Setup python.

```sh
update-alternatives --install /usr/bin/python python /usr/bin/python3 1
```

Set the locale, selecting `en_US`. This is needed by `asdf` to setup up nodejs and php.

```sh
dpkg-reconfigure locales
```

## 4. WSL Settings

- Create a user and add to the sudo group.
- Disable interop with the windows host, which will be done via `/etc/wsl.conf`.

```sh
adduser creativenull
usermod -aG sudo creativenull
```

Create or edit the `/etc/wsl.conf` file and add the following contents.
Ref [https://learn.microsoft.com/en-us/windows/wsl/wsl-config](https://learn.microsoft.com/en-us/windows/wsl/wsl-config).

```
[user]
default = [user name]

[interop]
enabled = false
appendWindowsPath = false

[boot]
systemd = true
```

Where `[user name]` is the user you want to create.

Now, logout and terminate the container and try to enter it (this is in powershell/windows terminal).

```sh
exit

# Below is in powershell once you exit
wsl --terminate Ubuntu
wsl -d Ubuntu
```

Perform a quick check with `whoami` and if it's the user you created then it was successful.

## 4. Dev Setup (WIP)

> [!IMPORTANT]
> Requires [dotfiles repo](https://github.com/creativenull/dotfiles.git)

```sh
git clone https://github.com/creativenull/dotfiles.git ~/dotfiles
```

### Shell

Using zsh and tmux.

```sh
sudo apt install -y zsh tmux
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
rm ~/.zshrc
ln -s ~/dotfiles/zshrc ~/.zshrc
ln -s ~/dotfiles/tmux.conf ~/.tmux.conf
```

Install optional artisan plugin for zsh.

```sh
git clone https://github.com/jessarcher/zsh-artisan.git ~/.oh-my-zsh/custom/plugins/artisan
```

Re-open terminal to see the changes.

### Core

Install `lsd` to replace `ls`.

```sh
cd ~
wget https://github.com/lsd-rs/lsd/releases/download/v1.0.0/lsd_1.0.0_amd64.deb
sudo dpkg -i lsd_1.0.0_amd64.deb
rm lsd_1.0.0_amd64.deb
```

Generate ssh key for Github, etc. Copy the `.pub` contents when create a new ssh key in Github settings.

```sh
ssh-keygen -t ed25519 -C [email]
```

Where `[email]` is your Github email.

### MYSQL

```sh
sudo apt install -y mysql-server
sudo systemctl enable mysql
sudo systemctl start mysql
```

Set a user and password give it all grants for local development (Not production).
Ref [https://www.digitalocean.com/community/tutorials/how-to-install-mysql-on-ubuntu-22-04](https://www.digitalocean.com/community/tutorials/how-to-install-mysql-on-ubuntu-22-04)

```sh
sudo mysql
```

```sql
CREATE USER 'creativenull'@'localhost' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON *.* TO 'creativenull'@'localhost' WITH GRANT OPTION;
FLUSH PRIVILEGES;
exit;
```

### Git

```sh
ln -s ~/dotfiles/gitconfig ~/.gitconfig
```

### asdf

Install asdf and plugins.

Follow the [zsh & git section](https://asdf-vm.com/guide/getting-started.html#_3-install-asdf) to setup asdf for zsh. Add this to `~/.zprofile` and NOT `~/.zshrc`.

```sh
git clone https://github.com/asdf-vm/asdf.git ~/.asdf --branch v0.13.1

# node
asdf plugin add nodejs https://github.com/asdf-vm/asdf-nodejs.git
asdf install nodejs 20.14.0
asdf global nodejs 20.14.0

# php
asdf plugin add php https://github.com/asdf-community/asdf-php.git
sudo apt install -y autoconf bison build-essential curl gettext git libgd-dev libcurl4-openssl-dev libedit-dev libicu-dev libjpeg-dev libmysqlclient-dev libonig-dev libpng-dev libpq-dev libreadline-dev libsqlite3-dev libssl-dev libxml2-dev libzip-dev openssl pkg-config re2c zlib1g-dev
asdf install php 8.2.20
asdf global php 8.2.20
```

### Deno

```sh
curl -fsSL https://deno.land/x/install/install.sh | sh
```

Add the export instructions to `~/.zprofile`.

### Editor

Install the essential editor tools before installing neovim.

```sh
sudo apt install -y ripgrep
```

Install packages for neovim build. Ref [https://github.com/neovim/neovim/blob/master/BUILD.md](https://github.com/neovim/neovim/blob/master/BUILD.md)

```sh
sudo apt-get install -y ninja-build gettext cmake unzip curl
```

Clone neovim into `~/.builds`. Specify the nvim version to checkout.

```sh
git clone https://github.com/neovim/neovim.git ~/.builds/neovim
cd ~/.builds/neovim
git checkout v0.10.0
```

Build the binary.

```sh
make CMAKE_BUILD_TYPE=Release
sudo make install
```

Finally, add my config.

```sh
ln -s ~/dotfiles/config/nvim ~/.config/nvim
```
