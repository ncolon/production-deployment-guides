tOps Prerequisites for Windows WSL

### This is for Windows Subsystem for Linux (WSL)

### First Install Ubuntu on Windows Subsystem for Linux (WSL) (if you have not done this before)
There are many ways and documents to enable WSL for windows. The below steps are written for Ubuntu shell. You can follow this link :https://ubuntu.com/wsl and scroll down to "Install Ubuntu on Windows Subsystem for Linux (WSL)"
Feel free to use other guides if you prefer

> #### Install Git
    sudo apt-get install git

Reference: https://docs.microsoft.com/en-us/windows/wsl/tutorials/wsl-git

**Install gh** 

```
curl -fsSL https://cli.github.com/packages/githubcli-archive-keyring.gpg | sudo dd of=/etc/apt/trusted.gpg.d/githubcli-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/trusted.gpg.d/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" | sudo tee /etc/apt/sources.list.d/github-cli.list > /dev/null
sudo apt update
sudo apt install gh
```
Reference: https://github.com/cli/cli/blob/trunk/docs/install_linux.md


___
___

> #### Install nvm (running below will install npm, no separate steps needed)

**Install cURL (a tool used for downloading content from the internet in the command-line) with:**

    sudo apt-get install curl


**Now Install nvm**

    curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash

Then run below three commands:
```
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"

```

To verify installation, enter: 

    command -v nvm 

The above should return 'nvm', if you receive 'command not found' or no response at all, close your current terminal, reopen it, and try again

___
___

> #### Install node

Install the current stable LTS release of Node.js (recommended for production applications): 

    nvm install --lts

Install the current release of Node.js (for testing latest Node.js features and improvements, but more likely to have issues): 

    nvm install node 

List what versions of Node are installed: 

    nvm ls 

Based on above you should see the two versions that you just installed listed

Individually node, npm, nvm versions can be checked by 

```
node –version
npm --version
nvm --version

```


Reference: https://docs.microsoft.com/en-us/windows/dev-environment/javascript/nodejs-on-wsl


___
___

> #### Install jq

```
sudo apt update
sudo apt install jq

```

https://www.howtoinstall.me/ubuntu/18-04/jq/

___
___

> #### Install tree

```
sudo add-apt-repository "deb http://archive.ubuntu.com/ubuntu $(lsb_release -sc) main universe restricted multiverse"
sudo apt install tree
tree –version

```

___
___


> #### Install oc
Use these instructions to get the latest version of oc 
(https://docs.openshift.com/container-platform/4.7/cli_reference/openshift_cli/getting-started-cli.html)

Follow the instruction in the above URL and download latest oc version to “Downloads” folder of your windows (You can download to any folder in windows but below instructions are written assuming you will use Downloads folders. You can use below instructions as guidance to navigate to any folder your downloading the oc file to)

Once oc file downloaded go to wsl windows ubuntu terminal 

```
cd /
cd mnt
cd c
```

`pwd` should show /mnt/c
`ls -la` This should show all your windows folders


    cd Users
    cd 6X1234567
    
"6X1234567" is used as user folder name. You use your user folder name in Users folder

    cd Downloads
    pwd  


should show /mnt/c/Users/6X1234567/Downloads

```
ls oc-4.9.18-linux.tar.gz
tar xvzf oc-4.9.18-linux.tar.gz
```
you should see
    README.md
    oc
    kubectl

```
sudo -i (and enter your password)
echo $PATH

```

you should see something like
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin
Now we should copy oc and kubectl to /usr/local/sbin as that is the path shown in echo $PATH

```
cp oc /usr/local/sbin
cp kubectl /usr/local/sbin

```

Open a new Ubuntu shell and type 

`oc –help` (should see all oc commands)
`oc version` (should give you open shift version)
`kubectl version` (should give you kubectl version)


___
___

> #### Install kubeseal

```
wget https://github.com/bitnami-labs/sealed-secrets/releases/download/v0.16.0/kubeseal-linux-amd64 -O kubeseal
sudo install -m 755 kubeseal /usr/local/bin/kubeseal

```



