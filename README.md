# Basis for python-based research projects. 

Update on tf 1.15 on Python 3.6 when used to python 3.6 and  tf 2.6

https://github.com/tensorflow/tensorflow/issues/20444#issuecomment-422899096

# Usage

After properly setting up the repository, the python package and jupyter lab you just have to open a WSL terminal, optionally log into a remote system and run:

```
cd <your-repository>
lab
```

Then, open `localhost:8888` (or whatever port you set your Jupyter Lab to) in your browser and you can start coding!

Use git to syncronize code between local and remote machines. For large files like datasets and exported models use scp.

Write scripts and run them inside a `screen` for long trainings on the remote machine.


# Installation

```diff
! lines marked like this mean you cannot copy-paste them
! replace parts of the command with your username, repository or package name where indicated
! the exclamation mark and <> brackets are never part of a command or something you should write into a file
```

## Installing WSL2 on your Windows PC

The first step to take if you are new to working with Python and Jupyter Lab is to install Windows Subsystem for Linux on your machine:

https://docs.microsoft.com/en-us/windows/wsl/install

This will serve as a basis for everything else. Even if you are only working remotely it's the easiest way to connect to your Linux servers. 


### Fix screen on WSL 2
```
mkdir ~/.screen
chmod 700 ~/.screen
```

Add this to `~/.bashrc` (explaination below):

```
export SCREENDIR=$HOME/.screen
```

To do this, open your `.bashrc` file:

```
nano ~/.bashrc
```

Then scroll to the bottom of the file (use the page-down button to scroll fast) and add the line above:

```
export SCREENDIR=$HOME/.screen
```

After closing the editor (Ctrl+O, Enter, Ctrl+X), you have to logout+login or you can run this command:

```
source ~/.bashrc
```

## SSH setup

If you want to work remotely on a Linux server you probably want to connect via SSH. To make life easier, here are a few tips to simplify working with ss, scp, jupyter lab, tensorboard etc.


If your .ssh directory does not yet exist create it with this command:
```
mkdir ~/.ssh
```

Then, create a key pair specific for every server you want to connect to:

```
ssh-keygen -t ed25519
```

This tool asks you for a name first, so pick a short and descriptive nickname for the server and enter:

```diff
! ~/.ssh/<server nickname>
```

Then, press ENTER twice if you dont want to set a password protecting the key. If your computer is protected via BitLocker and a good password you probably don't need to set one.


The next steps depend on the server setup you are trying to connect to. The goal is to add the public key you just created to the `.ssh/authorized_keys` file *on the server*. First print out the public key by using cat (make sure you are printing the file ending with `.pub`!):

```diff
! cat ~/.ssh/<remote_server_name>.pub
```
Copy this output and somehow paste it into the `.ssh/authorized_keys` file on the server. If you have a username and password for the server, then you should already know how to connect via SSH and adding this line is simple:

(on the server)
```
mkdir ~/.ssh
nano ~/.ssh/authorized_keys
(paste file as new line with right click)
(Ctrl+O and ENTER to save)
(Ctrl+X to close nano)
```

Now you can edit the `.ssh/config` on your local WSL system and add an entry that looks like this:

```diff
! Host <server nickname>
!    HostName <actual server address>
!    User <username on the server>
!    IdentityFile ~/.ssh/<server nickname>
!    LocalForward 127.0.0.1:8889 localhost:8888
```

After setting up SSH like this, you can login to the server by simply typing


```diff
! ssh <server_nickname>
```

Also, you can easily transfer files from/to the server by using the scp command from WSL:

local -> server
```diff
! scp -r <local file name> <server_nickname>:<remote target filename relative to $HOME>
```

server -> local
```diff
! scp -r <server_nickname>:<remote target filename relative to $HOME> <local file name>
```

This command basically just works like `cp -r` and is very easy to use if you get used to it. 

## Software and Packages 

The following should be run everywhere you want to work with jupyter lab or run code. In most cases, you will run them both inside WSL and on every server.
The first few installations require admin access (sudo) since they install packages globally, so you might need to ask the admin of the server to run some of them for you.

### apt-get Packages

There are some packages that need to be installed for processing images and videos, as well as the python package manager pip.
If you don't have sudo priviledges, you have to ask your admin to install them for you.

```
sudo apt-get update && sudo apt-get upgrade
```

```
sudo apt-get install -y python3-pip 
sudo apt-get install -y ffmpeg python3-opencv
```

### CUDA and cuDNN

CUDA and cuDNN are required for tensorflow to connect to the GPU.
If you have a decent GPU locally follow these instructions to install CUDA on WSL: 

https://docs.nvidia.com/cuda/wsl-user-guide/index.html


If you are on a Linux server and have sudo priviledges, follow the regular setup instructions:

https://developer.nvidia.com/cuda-downloads

In both cases, follow these additional instructions to install cuDNN:

https://docs.nvidia.com/deeplearning/cudnn/install-guide/index.html

Lastly, there are some additional steps required to make CUDA work with tensorflow. Please read about them in the link below, but in summary they require you to add this to your `~/.bashrc` (explaination above):

```
export PATH=/usr/local/cuda/bin${PATH:+:${PATH}}
export LD_LIBRARY_PATH=/usr/local/cuda/lib64${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}
```

```
source ~/.bashrc
```

https://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html#mandatory-post


### Node.js

To install Node.js follow these instructions further down in the README.md file. (you are probably looking for the newest Version for Ubuntu)
https://github.com/nodesource/distributions/blob/master/README.md#installation-instructions

You probably also want to run this command, but be aware that this might break some things if you don't know how to use node/npm:

```
npm config set prefix '~/.local/'
```

### Github

Github no longer accepts passwords as authentication. This is how you have to connect to your github now. First create a new keypair:

```
ssh-keygen -t ed25519
```

As key name, you could enter:

```
~/.ssh/github
```

Finish the setup and print your new public key like this:

```
cat ~/.ssh/github.pub
```

Copy this public key and add it to your Github account settings (a good name for the key is the nickname of the computer you are currently working on):

https://github.com/settings/keys

Also add the following entry to your `~/.ssh/config`:

```
Host github.com
   HostName github.com
   IdentityFile ~/.ssh/github

```

After adding the key, you can clone any repository from your Github account like this:

```diff
! git clone git@github.com:<your github account name>/<repository name>.git
```

The following steps assume that you want to use Jupyter Lab to work on your repository. 

### Clean Jupyter notebooks before committing changes

Add the clean filter to `.git/config` with this command:

```
git config filter.clean_notebook.clean $PWD/clean_notebook.py
```

### Additional settings to the repository

Always make sure your `.gitignore` file is up to date!


### .bashrc

All custom python packages in your repository are not available to Python yet, because they are not in one of the folders python is looking for packages. This means we have to add it to our `.bashrc` file (explaination above). This will also set up some aliases for python, pip and jupyter lab. 

```diff

alias python=python3
alias pip="python3 -m pip"
alias lab="jupyter lab --ip=* --port=8888 --no-browser"

! export PYTHONPATH=$PYTHONPATH:$HOME/<your-repository-path-and-name>
```

```
source ~/.bashrc
```

### Python packages

Now we install some basic packages

```
pip install --upgrade pip
pip install matplotlib
pip install ruamel.yaml
pip install ffmpeg-python
pip install opencv-python
```

```
pip install tensorflow
```

### Jupyter Lab

```
pip install jupyterlab
pip install ipywidgets
```

**LOG out and log in now!**

```
exit
```

In order to properly install Jupyter Lab, run these additional commands:

### Juptyter lab git extension

To install and activate the git extension for the Jupyter lab, run the following commands:

```
pip install jupyterlab-git
```

```
jupyter server extension enable --user --py jupyterlab_git
```

### Jupyter lab language checks extension

```
pip install git+https://github.com/krassowski/python-language-server.git@main
pip install 'python-lsp-server[all]'
```

```
pip install jupyterlab-lsp
```

```
jupyter server extension enable --user --py jupyter_lsp
```

By default, there are many warnings regarding some pycodestyle checks that just don't make sense in a Notebook file. You can suppress some warnings by creating this file:

```
mkdir ~/.config
nano ~/.config/pycodestyle
```

It can look something like this:

```
[pycodestyle]
ignore = E261, W293, E409, E701, E402
max-line-length = 120
```

### diff/merge support for notebooks

```
pip install nbdime
```

```
nbdime extensions --enable
```

And for the repository, enable the git integration:

```diff
! cd <your-repository-name>
```

```
nbdime config-git --enable
```

## Additional Settings inside Jupyter Lab

To run Jupyter Lab, you have to either run the full command or use the alias we set earlier:

Make sure you are starting lab in the root directory of your repository!

```
lab
```

Open `localhost:8888` or `localhost:8889` in your browser and finish setting up Jupyter Lab like this:

### Minor adaptations

Settings -> Theme -> JupyterLab Dark


Settings -> Save Widget State Automatically


### Add collapsible headers extension inside Jupyter Lab

Adding this extension will require Node.js. Search for `collapsible` in the extension mamager (puzzle icon on the left) and click `Install` instead of clicking the title. It will then ask for a rebuild. Confirm it and wait for completion. Then click Save&Reload. 


Run Jupyter Lab on remote server on local computer
https://ljvmiranda921.github.io/notebook/2018/01/31/running-a-jupyter-notebook/

