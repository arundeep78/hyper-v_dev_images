# Hyper-V image : Debian 11 base image

## [Download Debian 11](https://www.debian.org/CD/http-ftp/#stable) 

from its mirror servers for your architecture and specific situation. In my case it Debian 11 network install. It is small and I did not need graphic frontend for my usage.

## Install Debian on VirtualBox. 

If you need help, you can use this [simple procedure from MS](https://docs.microsoft.com/en-us/virtualization/hyper-v-on-windows/quick-start/quick-create-virtual-machine)
   1. make sure to select appropriate location for your VM. Default is on C: drive, which may not be enough.
   2. Configure RAM and CPU based on your machine's capacity and requirement.
   3. Make sure to confiure a **non-root user and set-up sudo** for it.


## Configure Windows terminal profile to connect to VM machine.

Configure Windows Terminal profile to ssh to VM machine. You need to configure "command line" = ssh user@ip4address.
Set other values to as per your needs. e.g. I use [Blazer color scheme](https://windowsterminalthemes.dev/?theme=Blazer) for Windows Terminal, same as for WSL images.

## Install ZSH

1. Install ZSH and set it as your default shell

   ```zsh
   sudo apt install zsh -y
   chsh -s $(which zsh)
   ```

2. Logout, close session and login in again to configure ZSH

## Install curl

In case it is not installed by the default selection.

```zsh
sudo apt update
sudo apt install curl
```

## Git Setup

### Install Git in Debian

  Git needs to be installed on each VM.

  ```zsh
  sudo apt install git -y
  ```

## install GCM in Debian

Follow instructions from [Git Credentials Manager github repository](https://github.com/GitCredentialManager/git-credential-manager/blob/main/README.md#linux-install-instructions) for Linux.

```zsh
wget /path_to_latest.deb_package
sudo dpkg -i <path-to-package>
git-credential-manager-core configure
```

## Install Github CLI

To know more, read [official documentation](https://cli.github.com/) from GitHub. [Linux specific installation instructions](https://github.com/cli/cli/blob/trunk/docs/install_linux.md) are available at github.

```zsh
curl -fsSL https://cli.github.com/packages/githubcli-archive-keyring.gpg | sudo dd of=/usr/share/keyrings/githubcli-archive-keyring.gpg

echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" | sudo tee /etc/apt/sources.list.d/github-cli.list > /dev/null

sudo apt update

sudo apt install gh
```

## Install and configure Oh My ZSH

Rather than me saying anything about it, just read on [Oh My ZSH website](https://ohmyz.sh/). Atleast for me, with its configurable plugins and themes, it made working on Linux shell easy and intersting. I am not one of those geeks who like to remember every command and like black an white screen!

1. Install pre-requisities. if you have followed all the steps above then pre-requistes are already installed.
2. Install Oh My ZSH

   ```zsh
   sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
   ```

3. [Install Powerlevel10K theme](https://github.com/romkatv/powerlevel10k)
   1. [install recommended font](https://github.com/romkatv/powerlevel10k#meslo-nerd-font-patched-for-powerlevel10k) - Meslo Nerd font
   2. Install powerlevel10k theme

   ```zsh
      # Clone the repository
      git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k
      ```

   3. Set `ZSH_THEME="powerlevel10k/powerlevel10k"` in `~/.zshrc`.
   4. Logout and login into the shell to activate it. Or you can also execute `p10k configure` to start the configuration wizard. Follow your preferences and configure the prompt. I went for `rainbow` style.
   

5. Install [zsh auto-suggestions](https://github.com/zsh-users/zsh-autosuggestions). Based on the installation type procedure may change. For Oh My ZSH method is below.  

   1. Copy the plugin

      ```zsh
      git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions 
      ```

   2. Add plugin in the `.zshrc` file

      ```zsh
      plugins=( 
         # other plugins...
         zsh-autosuggestions
         )
      ```

   3. Based on your terminal color layout, you might have to configure parameter `ZSH_AUTOSUGGEST_HIGHLIGHT_STYLE`. For Oh My ZSH, you can edit that parameter in `$ZSH_CUSTOM/plugins/zsh-autosuggestions/zsh-autosuggestions.zsh`. 

6. Configure [plugins for Oh My ZSH](https://github.com/ohmyzsh/ohmyzsh/wiki/Plugins). Add below plugins to `~./zshrc` to enable autocompletions
y
   ```zsh
   plugins=(
          # other plugins...
          zsh-autosuggestions
          git 
          gh
        )
   ```


## Install SNAP package manager

```zsh
sudo apt update
sudo apt install snapd
sudo snap install core
```

### Configure ZSH to work with SNAP applications

This is something I learned later, when I faced issues with microk8s. but it is better to put it here for others not to struggle.

Apparently [SNAP some is not able to setup $PATH for its applications in ZSH shell](https://forum.snapcraft.io/t/supporting-the-zsh/2264). I do not know the reason or challenges, but it seems to be really old problem and still is.

**Solution** is to configure below line in `/etc/zsh/zshenv` file.
There are debates around which file it should be added e.g. `.zshrc`, `/etc/zsh/zprofile` and others. I selected zenv as this is where other one of the basic path parameter is configured. 
```zsh
emulate sh -c 'source /etc/profile.d/apps-bin-path.sh'
```

## Remote SSH development using Visual Studio code

1. [Install Visual Studio Code](https://code.visualstudio.com/download) in Windows.
2. [Install Remote Development Extension pack](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.vscode-remote-extensionpack). If you are not going to use docker or WSL then you can only install Remote SSH extension.
3. Install [Supported SSH client](https://code.visualstudio.com/docs/remote/troubleshooting#_installing-a-supported-ssh-client) for your OS where VS Code is installed. In my case it is Windows 11 and there is an inbuilt SSH client already installed.
4. Install Open SSH server on Debian. If you have followed steps so far, it is already installed.
5. Run VS Code in windows and from Command pallete (Ctrl+Shift+P) execute `Remote-SSH: Connect to Host` and enter username@ip_or_host and select `OS type`. In this case it is `Linux`
   ![Configure Remote SSH host in VS Code](images/base_image/base_connect_ssh_host.drawio.svg)
6.  Enter password. VS code will make and SSH connection, install VS code on the server and start a session in Hyper-V Debian VM.
  ![VS Code session in Remote SSH on Virtual Box](images/base_image/base_remote_ssh_connection.drawio.svg)
7. Now one can open any folder in the Remote system using **File > Open**
8. Install any VS code remote extension for your requirements e.g. `markdown`, `python` etc.
9. Configure SSH keys to enable password-free access (Optional). Follow instructions from [official documentation](https://code.visualstudio.com/docs/remote/troubleshooting#_quick-start-using-ssh-keys) for your platform. 

## Hyper-V VM with Debian 11 is ready!!

One can backup the virtual machine .vhdx file or the whole folder. This file can then later be imported to start on a different machine or create another VM based on this image.


One drawback here I feel is the size of the VDI file. 
- This VDI file is 3.2GB in size!! 
- Even when I did and export as an .OVA file its size was 1.2GB
- in comparison to only a 300MB export of WSL image.

I do not know if that is due to different Debian version that got installed with WSL vs the one I downloaded for Virtual Box. But, if this Virtual Box installation helps me to get going with Kubernetes and airflow installations, this space is not a big deal then. Now, lets create next image