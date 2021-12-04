# NGS_WSL2
Here I show the commands I executed setting up Windows Subsystem for Linux 2 (WLS2). I've followed the instructions on [Best practices for setting up a WSL development environment](https://docs.microsoft.com/en-us/windows/wsl/setup/environment), where you can find recommended tutorials for every step.

## Prerequisites to install WSL2 
Prerequisites: Windows 10 version 2004 and higher (Build 19041 and higher) or Windows 11. Check your PC informations on settings > system > about or search "about your PC" in the Search the web and Windows field.

I am using an IdeaPad Gaming 3i Lenovo:
- processor Intel(R) Hexa-Core(TM) i7-10750H CPU @ 2.60GHz
- x64-based processor
- 16.00 GB RAM 
- SSD 512GB 
- NVIDIA GeForce GTX 1650 4GB
- Windows 11 version 21H2
- OS build 22000.318

You can register to [Windows Insider Program](https://insider.windows.com/en-us/getting-started) to get Windows 11.

## WSL2 installation
First I tried to use the comand `wsl --intall`, which should install everything required to run WSL. But it didn't work out for me, so I followed the instructions in the microsoft documentation page to manual installation steps [docs.microsoft.com/en-us/windows/wsl/install-manual](https://docs.microsoft.com/en-us/windows/wsl/install-manual).
### Step 1 - Enable the Windows Subsystem for Linux
Open PowerShell as Administrator (Start menu > PowerShell > right-click > Run as Administrator) and enter the following command:
```PowerShell
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
```
### Step 2 - Enable Virtual Machine feature
Open PowerShell as Administrator (Start menu > PowerShell > right-click > Run as Administrator) and enter the following command:
```PowerShell
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
```
### Step 3 - Restart your machine to complete the WSL install and update to WSL 2.
### Step 4 - Download the Linux kernel update package
1. Download the latest package:
[WSL2 Linux kernel update package for x64 machines](https://wslstorestorage.blob.core.windows.net/wslblob/wsl_update_x64.msi)
2. Run the update package downloaded. (Double-click to run - you will be prompted for elevated permissions, select ‘yes’ to approve this installation.)
### Step 5 - Set WSL 2 as your default version
Open PowerShell as Administrator (Start menu > PowerShell > right-click > Run as Administrator) and enter the following command:
```PowerShell
wsl --set-default-version 2
```
## Ubuntu 20.04 LTS installation
1. Open the Microsoft Store and search for Ubuntu 20.04 LTS.
2. From the distribution's page, select "Get".
3. Launch the newly installed Linux distribution, a console window will open and you'll be asked to wait for a minute or two for files to de-compress and be stored on your PC. You will then need to create a user account and password for your new Linux distribution.
4. Update and upgrade packages running the following command on Ubuntu 20.04:
```bash
sudo apt update && sudo apt upgrade
```

## Windows Terminal installation
1. Open the Microsoft Store and search for Windows Terminal.
2. From the distribution's page, select "Get".
3. After installation, when you open Windows Terminal, it will start with the PowerShell command line as the default profile in the open tab. To change the default profile:
(i) Open Windows Terminal and go to the Settings window.(ii) Select Startup and choose the Ubuntu 20.04.

## File storage
You should store your project files on the same operating system as the tools you plan to use.
To open your WSL project in Windows File Explorer, enter the following command on Ubuntu 20.04 (do not forget the last dot).
```bash
explorer.exe .
```
The default starting directory for the Ubuntu WSL Terminal is `/mnt/c/Users/<UserName>` (replacing <UserName> with your user name), it is better to change for the Linux file system. All you have to do is open the Windows Terminal settings, select Ubuntu 20.04 LTS, and change the default starting directory for `\\wsl.localhost\Ubuntu-20.04\home\<UserName>` (replacing <UserName> with your user name)
If you need to access the Windows file directory from your WSL distribution command line, the directory would be accessed using `/mnt/c/Users/username`.

## VSCode installation
1. Download [Visual Studio Code](https://code.visualstudio.com/download) on Windows file system (not in your WSL file system).
2. Run the package downloaded in the previous step, be sure to check **Add to PATH** when asked to Select Additional Tasks during installation.
3. Open VSCode and install the Remote Development extension pack.

![Remote Development extension installation](https://drive.google.com/uc?export=view&id=1zqerkiw2RsVD8jTFy7v4WXfjg2babQFI)

4. Once VS Code is installed and set up, you can open your WSL project with a VS Code remote server: open a command line for Ubuntu 20.04 and `code .`.

## Git installation
1. If you don't yet have a GitHub account, you can sign-up for one on [GitHub](https://github.com/).
2. Git comes already installed on Ubuntu 20.04 LTS, however, you may want to update to the latest version. For the latest stable Git version in Ubuntu, open a command line for Ubuntu 20.04 and enter the comand:
```
sudo apt-get install git
```
3. Set your name with this command (replacing "Your Name" with your GitHub username):
```
git config --global user.name "Your Name"
```
4. Set your email with this command (replacing "youremail@domain.com" with your GitHub email):
```
git config --global user.email "youremail@domain.com"
```
If you need to edit your Git config, you can do so with a built-in text editor like nano: `nano ~/.gitconfig`.
5. To set up GCM Core for use with a WSL distribution, enter this command:
```
git config --global credential.helper "/mnt/c/Program\ Files/Git/mingw64/libexec/git-core/git-credential-manager-core.exe"
```
6. Generate a new SSH key with the command below (replacing "youremail@domain.com" with your GitHub email).
```
ssh-keygen -t ed25519 -C "your_email@domain.com"
```
When you're prompted to "Enter a file in which to save the key," press Enter. At the prompt, you can type a secure passphrase and press enter. Or just press enter for no passphrase.
7. Enter `cd ~/.ssh`. Then, enter `cat id_ed25519.pub`. Copy your key (from the "ssh-ed25519" to the e-mail).
On your browser, go to `github.com/settings/ssh/new` and paste your key on the key field, don't add any newlines or whitespace. Type a title and press "add SSH key".

## Docker installation
1. Download [Docker Desktop](https://docs.docker.com/desktop/windows/wsl/) and follow the installation instructions.
2. When Docker Desktop restarts, ensure that **"Use the WSL 2 based engine"** is checked in **Settings > General**. 
3. Go to Settings > Resources > WSL Integration. Enable Ubuntu-20.04. Click "Apply & Restart".
4. You can get familiar with docker by following the Docker getting-started tutorial. Open DockerDesktop, then open a command line for Ubuntu and run `docker run -d -p 80:80 docker/getting-started`. Open "localhost:80" on your browser, you will see the page below, then follow the instructions.

![Docker GPU test](https://drive.google.com/uc?export=view&id=14-tS4orvKNl_MaFIIi5MOy4Jbj8UDsk0)

## SQLite installation
Open a command line for Ubuntu 20.04 and enter the command:
```
sudo apt install sqlite3
```

## Set up GPU acceleration for faster performance
If you don't have a NVIDIA GPU, skip this part.
1. Download and install [NVIDIA supporting WSL 2 GPU Paravirtualization driver](https://developer.nvidia.com/cuda/wsl/download)
2. Install NVIDIA Container Toolkit. Open a command line for Ubuntu 20.04:

a) Set up the stable repositories and the GPG key. The changes to the runtime to support WSL 2 are available in the stable repositories, enter the commands:
```
distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
```
```
curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add -
```
```
curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | sudo tee /etc/apt/sources.list.d/nvidia-docker.list
```

b) Install the NVIDIA runtime packages (and their dependencies) after updating the package listing, enter the commands:
    
```
sudo apt-get update
```
```
sudo apt-get install -y nvidia-docker2
```
3. Restart your machine
4. To validate that everything works as expected, open Docker Desktop to start the engine, then open a command line for Ubuntu 20.04 and run a short benchmark on your GPU:
```
docker run --rm -it --gpus=all nvcr.io/nvidia/k8s/cuda-sample:nbody nbody -gpu -benchmark
```
Few seconds after running the command you should visualize something like this on your terminal:

![Docker GPU test](https://drive.google.com/uc?export=view&id=1--_mUi2x7Le8VbDt0OLsWzcq41bpj-_Z)

You can also visualize the image on Docker Desktop:

![Docker GPU test](https://drive.google.com/uc?export=view&id=12yrtHPNgVjKcea6N6K4vJU4uVcwvZE-Y)
