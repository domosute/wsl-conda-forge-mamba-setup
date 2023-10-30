# WSL Conda Forge Mamba Setup

## 1. Motivation
Due to the increase of packages, Anaconda3 installer is becoming painfully slow. This step-by-step tutorial is focusing on Conda Forge based package installer and mamba CLI package manager for speed-optimized package installation/maintenance process.

## 2. Installation

### 2.1. Open Powershell as an Administrator

### 2.2. Enable WSL subsystem
```
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Windows-Subsystem-Linux
```
### 2.3. Install Ubuntu
From Microsoft Store, select Ubuntu and install it and Launch.
Pin it to Task bar as well.

### 2.4. Create user account
Align with windows username. Set password accordingly.    
_In case if the default user name was not created during installation process, perform `ubuntu config --default-user <username>` to create it._

### 2.5. Update and install packages
```
$ sudo apt update
$ sudo apt upgrade
$ suto apt-get upgrade openssl
$ sudo apt-get install -y apt-utils nodejs npm
$ sudo apt-get install -y pandoc poppler-utils emacs-nox
$ sudo apt-get install -y tree
```
_[Note]: upgrade openssl to make sure `conda update` to work..._

### 2.6. (Optional) Create `/dfs` directory.
#### 2.6.1. Create Directories for Mappping
Map if network drive is available. Below example, P: and Z: drives are exisited for the Windows workstation.
```
$ sudo mkdir -p /dfs/{p,z}
```
### 2.6.2. Auto mount network drive
Make sure to set parameter in `/etc/wsl.conf` so that the drive is mounted during startup.
```
[automount]
enabled=true
mountFsTab=true
```

### 2.6.3. Edit /etc/fstab for mounting P: Z: drive
```
LABEL=cloudimg-rootfs   /        ext4   defaults        0 1
P: /dfs/p drvfs defaults 0 0
Z: /dfs/z drvfs defaults 0 0
```
_[Note]: 2021/05/12: If the network drives lost mount, manual mount or restart is required._
```
$ sudo mount -a
```

### 2.7. Download Mamba-Forge Version of Installer (miniforge)
Install it under /home/`<username>`/mamba directory.    
https://mamba.readthedocs.io/en/latest/installation.html

```
$ cd ~
$ curl -L -O "https://github.com/conda-forge/miniforge/releases/latest/download/Mambaforge-$(uname)-$(uname -m).sh"
$ bash Mambaforge-$(uname)-$(uname -m).sh
```
_[Note]: After installation exit Ubuntu and restart._

If the environment file is already available, populate with `mamba env update` command.
```
$ mamba env update -n base -f saved_env_file.yml
```

### 2.8. Install Packages for Data Analytics
```
$ mamba update mamba
$ mamba update -y --all
$ mamba install -y jupyter jupyterlab numpy scipy pandas nodejs
$ mamba install -y xlrd xlsxwriter
```
_[Optional]: Depending upon the script running, proper packages needed to be installed with `mamba install` commmand_

### 2.9. Add SSL Certificate
This will enable to pull the repository with git command.
```
openssl s_client -showcerts -servername www.github.com -connect www.github.com:443 </dev/null 2>/dev/null | sed -n -e '/BEGIN\ CERTIFICATE/,/END\ CERTIFICATE/ p'  > github.pem
cat github.pem | sudo tee -a /etc/ssl/certs/ca-certificates.crt
```
_[Note]: Disable SSL verification if above does not work..._
```
git config --global http.sslverify false
```

### 2.10. Set Brower to Auto Start While Running `jupyter lab`
Create jupyter notebook config file.
```
jupyter lab --generate-config
```
Edit `~/.jupyter/jupyter_lab_config.py`, modify the following.
```
#  Disabling this setting to False will disable this behavior, allowing the
#  browser to launch by using a URL and visible token (as before).
#  Default: True
# c.NotebookApp.use_redirect_file = True
c.ServerApp.use_redirect_file = False
```

Define browser for auto start.
- Chrome
```
c.ServerApp.browser = u'/mnt/c/Program\ Files/Google/Chrome/Application/chrome.exe %s'
```
- Firefox
```
c.ServerApp.browser = u'/mnt/c/Program\ Files/Mozilla\ Firefox/firefox.exe %s'
```

### 2.11. Run jupyter lab
```
$ jupyter lab &
```

## 3. Optional Setup

### 3.1. Scheduler (cron)
This is for the user to wish run the script in periodical manner.    

#### 3.1.1. installation 
Install cron if it is not already installed.
```
$ sudo apt-get install -y cron
```

Modify Sudoers file to run cron during WSL startup.

```
$ sudo vi /etc/sudoers
```

Add the following at the end of sudoers file. Modify the file by existing `wq!`.
```
# Start cron during startup
%sudo ALL=NOPASSWD: /etc/init.d/cron start
```
_[Note]: 09/22/2022: This does not start cron for Windows 10 environment._
Workaround:
1). Insert at the end of user's .bashrc
```
# Running cron at the startup
sudo service cron start
```
2). set user's account so that password won't be required with sudo command.    
_Modify <username> according to the setup._
```
$ echo "<username> ALL=(ALL) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/<username>
```

Restart Ubuntu to make sure cron will work.    
```
$ sudo service cron status
* cron is running
```
#### 3.1.2. Setting Up Cron job
Below is the template to run ipython notebook `test.ipynb` which is saved under `/home/<username>/notebook/test/` directory every 3:45pm except Saturday and Sunday.

1). Edit crontab
```
$ crontab -e
```

Add below, this will add time stamp in cron_job.log file under user's directory every minute.    
Once it is verified, remove the job.

```
45 15 * * 1-5 /home/<username>/notebook/test/test_schedule.sh
```

2). Create `test_schedule.sh` in /home/<username>/notebook/test/ directory as below.    
_[Note]: Placing ipython command directly in crontab doesn't really work..._
```
#!/bin/sh
cd /home/<username>/notebook/test
/home/<username>/mamba/bin/ipython -c "%run test.ipynb"
cd ~
```

3). Make sure to set the `test_schedule.sh` executable.
```
$ chmod +x /home/<username>/notebook/test/test_schedule.sh
```

4). Cron Job Log will be shown in `/var/log/cron.log`. If it is not shown, reinstall rsyslog and reconfigure it.
```
$ sudo apt-get purge -y rsyslog
$ sudo apt-get install -y rsyslog
$ sudo dpkg-reconfigure rsyslog
```
