# A Joomla Development Setup

## Base Environments

- Ubuntu-20.10 (recommended)
- Ubuntu-20.04 over WSL2, **Windows 10/11**
- Windows 10/11

### Setup Routes

- Linux/WSL2 on Windows

  1. [LAMP][lamp]: Natively setup your stack on Linux. Click [here][lamp-setup-guide] for setup guide.

     <details>
         <summary>Expand setup instructions!</summary>


     ```bash
     git clone "https://github.com/teddysun/lamp.git"
     cd lamp
     sudo ./lamp.sh --apache_option 1 --db_option 8 --php_option 6 --kodexplorer_option 2 --apache_mod
     ules mod_wsgi,mod_security --php_extensions apcu,xdebug --db_manage_modules phpmyadmin
     ```

     </details>

     

  2. Docker or Podman on **Ubuntu-20.10** or **Ubuntu-20.04 WSL2**. Click [here][podman-setup-guide] for setup guide.

     <details>
         <summary>Expand setup guide!</summary>


     ### Setting up an Environment with Podman

     [Podman][podman] is a daemonless container engine for working with OCI Containers on Linux systems. In practice, it can be used as a drop-in replacement for [Docker][docker]. If you want to set up your environment with Docker instead, feel free to replace the installation of Podman with Docker.

     Some of the additions to `~/.bashrc` are only required for WSL2. They may be dropped if you're in a native Linux environment. 

     1. [Install Podman](https://podman.io/getting-started/installation.html)

     2. Get Docker Compose:

        ```bash
        sudo curl -L "https://github.com/docker/compose/releases/download/1.25.4/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
        sudo chmod +x /usr/local/bin/docker-compose
        ```

     3. Additions to your `~/.bashrc`:

        ```bash
        # set $XDG_RUNTIME_DIR for podman [only WSL2!]
        if [[ -z "$XDG_RUNTIME_DIR" ]]; then
        	export XDG_RUNTIME_DIR=/run/user/$UID
        	if [[ ! -d "$XDG_RUNTIME_DIR" ]]; then
        		export XDG_RUNTIME_DIR=/tmp/$USER-runtime
        		if [[ ! -d "$XDG_RUNTIME_DIR" ]]; then
        			mkdir -m 0700 "$XDG_RUNTIME_DIR"
                 fi
        	fi
        fi
         
        # alias docker to podman
        alias docker=podman
        # custom DOCKER_HOST to work with docker-compose
        export DOCKER_HOST="unix://${XDG_RUNTIME_DIR}/podman/podman.sock"
        
        # get local IP address [only WSL2!]
        get_local_ip_wsl2() {
        	ip addr | grep 'eth0' | grep -Po '(?<=inet )[0-9\.]*(?=/)'
        }
        ```

     4. Podman services that run in both native and WSL2 environments. Need to be started on startup.
        If you're not in a WSl2 environment, you can avoid using these services by enabling the `podman.socket` systemd service with `sudo systemctl enable podman.socket`

        ```bash
        nohup podman system service --time=0 < /dev/null > /dev/null 2>&1 &
        # second service -- use only if you're on WSL2 and run IDE on Windows [onyl WSL2!]
        nohup podman system service --time=0 "tcp:$(get_local_ip_wsl2):8089" < /dev/null > /dev/null 2>&1 &
        ```

        </details>

- Windows Natively

  1. WAMP, XAMPP or EasyPHP. Click [here][jdocs-environment] for links and setup guides on Joomla! Docs

### Setup Your Joomla! Installation
#### Clone Joomla

```bash
git clone "https://github.com/joomla/joomla-cms.git" -b 4.1-dev
```
#### Build Joomla
- Install PHP dependencies
   ```bash
   composer install
   ```
- Install JavaScript Dependencies, Build JS and CSS
   ```bash
   npm ci
   ```

#### Install Joomla!
- Go to your localhost (port depends on how you set up your environment)
- Follow Joomla! installation instructions.

#### Install Joomla! Patch Tester (Joomla 4.1)
With the Patch Tester, you can easily apply changes from Pull Requests and test them. Alternatively, this can also be done using plain Git (and the GitHub CLI client), however we'll set this up here as this is the easiest and recommended way to get started:
- Log in to the Joomla! Backend: https://localhost/administrator
- Go to the System Menu
- Go to Extensions under the Install module
- Go to the Install from URL Tab and enter: `https://github.com/joomla-extensions/patchtester/releases/download/4.1.0/com_patchtester_4.1.0.zip`
- Click on Check & Install and follow instructions.

[lamp]: https://github.com/teddysun/lamp
[podman]: https://podman.io/
[jdocs-environment]: https://docs.joomla.org/Setting_up_your_workstation_for_Joomla_development
[lamp-setup-guide]: #setting-up-an-environment-with-lamp-on-linux

[docker]: https://docker.com