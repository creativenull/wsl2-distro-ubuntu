# Custom Ubuntu Distro for WSL2

This is a quick repo for me to build out my dev environment in WSL2. Planning to create scripts based on the instructions in here.

Adapted from [https://learn.microsoft.com/en-us/windows/wsl/use-custom-distro](https://learn.microsoft.com/en-us/windows/wsl/use-custom-distro)

## Requirements

- Docker for Windows
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

## 2. Import to WSL2

```pwsh
wsl --import Ubuntu C:\path\to\install\dir C:\path\to\ubuntu.tar
```

Re-open windows terminal and check if you can enter the container.

```pwsh
wsl -d Ubuntu
```

If you enter into the root shell then it's successfully installed.

## 3. Container setup

Setup the container for the user, while inside the container.

> [!IMPORTANT]
> We are inside the container now and not in powershell

Uncompress the container since it was a docker container before, we want the full ubuntu experience now.

```sh
unminimize
```

Install core tools for the dev environment.

```sh
apt install -y nano vim curl wget unzip zip git man-db locales locate build-essential autoconf sudo python3 python3-pip
```

Setup python.

```sh
update-alternatives --install /usr/bin/python python /usr/bin/python3 1
```

Set the locale to your region, selecting `en_US` for mine. This is needed by `asdf` node plugin to setup up nodejs.

```sh
dpkg-reconfigure locales
```

## 4. WSL Settings

Need to setup a user and disable interop with the windows host. This can be done via `/etc/wsl.conf` file in the container.

Create a user first and add to the sudo group.

```sh
adduser creativenull
usermod -aG sudo creativenull
```

Edit/create the `/etc/wsl.conf` file and add the following contents. Ref [https://learn.microsoft.com/en-us/windows/wsl/wsl-config](https://learn.microsoft.com/en-us/windows/wsl/wsl-config).

```
[user]
default = creativenull

[interop]
enabled = false
appendWindowsPath = false
```

Now, logout and terminate the container and try to enter it.

```
wsl --terminate Ubuntu
wsl -d Ubuntu
```

Perform a quick check with `whoami` and if it's the user you created then it was successful.

## 4. Dev Setup (WIP)

### Editor tools

Install the essential editor tools before installing neovim.

```sh
sudo apt install -y fzf ripgrep
```

Install packages for neovim build. Ref [https://github.com/neovim/neovim/blob/master/BUILD.md](https://github.com/neovim/neovim/blob/master/BUILD.md)

```sh
sudo apt-get install -y ninja-build gettext cmake unzip curl
```

Clone neovim into `~/.builds`. Specify the nvim version to checkout.

```sh
git clone https://github.com/neovim/neovim.git ~/.builds/neovim
cd ~/.builds/neovim
git checkout v0.9.5
```

Build the binary.

```sh
make CMAKE_BUILD_TYPE=Release
sudo make install
```
