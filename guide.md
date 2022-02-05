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

### Cloning Joomla

```bash
git clone "https://github.com/joomla/joomla-cms.git" -b 4.1-dev
```

## Setting up an Environment with LAMP on Linux




[lamp]: https://github.com/teddysun/lamp
[podman]: https://podman.io/
[jdocs-environment]: https://docs.joomla.org/Setting_up_your_workstation_for_Joomla_development
[lamp-setup-guide]: #setting-up-an-environment-with-lamp-on-linux

[docker]: https://docker.com