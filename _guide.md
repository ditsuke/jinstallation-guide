# Setting up A Development Environment For Joomla!

## Operating Systems/Base Environments Covered

- Windows 10/11
- Ubuntu-20.04 over WSL2, **Windows 10/11**
- Ubuntu-20.10

## Preparing an Environment
Written primarily in PHP on the backend, Joomla! requires the PHP interpreter along with a web server to render and serve the web interface over HTTP/S. On the frontend, it extensively uses CSS for styling and JavaScript for functionality and improved UX. Recently there's also been a slow transition moving some parts of the Joomla! web interface over to the VueJS JavaScript Framework.

While a pre-built package would only require the interpreter and web server, Joomla! development also requires some other tools to aid the development process, manage dependencies and compile/transpile its web assets. These tools include Composer to manage PHP dependencies and NPM to manage JavaScript dependencies and build tools to transpile/compile our JS and CSS.

As of Joomla! 4.1, development and usage require:
| Tool      | Recommended  | Minimum   |
| ----      | -----------  | -------   |
| PHP       | 8.0          | 7.2.5     |
| MySQL     | 5.6+         | 5.6       |
| Apache    | 2.4+         | 2.4       |
| Composer  | `N/A`        | `N/A`     |
| NPM       | `N/A`        | `N/A`     |

Note that we also support other databases (**PostgreSQL**) and web servers (**Nginx**, **MS IIS**). Drop-in MySQL replacements like **MariaDB** and **Percona** are also widely in use and generally work just as well as MySQL, although not _officially_ supported (See [#14141][joomla-mariadb-official-support-thread] for a discussion).


- **Linux/WSL2 on Windows**

  1. [LAMP][lamp]: Natively setup your stack on Linux.
  The Apache, MySQL and PHP (or Perl) stack on Linux is popularly known as the LAMP stack, which just so happens to be the stack Joomla! runs on. To set this stack up, we can use the handy namesake "LAMP" script which takes care of not only installing the dependencies but also some basic configuration and such.
     <details>
         <summary><u>Expand setup guide</u> 🔴👈</summary>

     ```bash
     git clone "https://github.com/teddysun/lamp.git"
     cd lamp
     # install the lamp stack
     sudo ./lamp.sh --apache_option 1 --db_option 8 --php_option 6 --kodexplorer_option 2 --apache_mod
     ules mod_wsgi,mod_security --php_extensions apcu,xdebug --db_manage_modules phpmyadmin
     # install composer and npm
     sudo apt-get install npm composer -y
     ```

     </details>

     

  2. Docker or Podman on **Ubuntu-20.10** or **Ubuntu-20.04 WSL2**.
  Docker and Podman are tools to create and manage containers. These containers
  can have highly controlled environments made by using static filesystem images which are then made into functional Linux containers.  
   While these containerisation technologies best work on UNIX systems, they're also trivially set up on Windows systems through WSL2 (recommended) or HyperV.
     <details>
         <summary><u>Expand setup guide</u> 🔴👈</summary>

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
      5. Once you clone the Joomla! repo:
         0. `cd` to the Joomla! directory.
         1. Copy the `docker-compose.yml` file from here to the folder.
         2. Spin up the Podman services: `docker-compose up -d`
         3. Visit http://localhost:8012 to check if things work (you should find Joomla! here, served by the `joomla-dev` container).

        </details>

- **Windows (native)**

  1. WAMP, XAMPP or EasyPHP. Click [here][jdocs-environment] for links and setup guides on Joomla! Docs.
      - Install `composer` and `npm`. This is easiest done through the [Scoop][scoop] package manager for Windows.

### Preparing Your Joomla! Installation
We have covered the environment required to develop Joomla! The next stop: Joomla! itself.

Joomla! Docs has an excellent resource on setting up an environment that covers much more ground that we will here and includes troubleshooting for problems that you may across. Click [here][jdocs-env] to access.

#### Cloning or Getting Your Own Copy of Joomla!
Joomla! core development uses Git as a Version Control System and is hosted over on GitHub. To get started, you need to know only the basics of how the Git VCS works. We also use GitHub Issues and Pull Requests to collaborate and track issues and development activity, which becomes important as you become more involved with the project.

To get a local copy of the Joomla 4.1 development branch:

```bash
git clone "https://github.com/joomla/joomla-cms.git" -b 4.1-dev
```
#### Building Joomla
When you get the core Joomla! codebase via Git, it is missing external dependencies and also requires building of certain assets before it can work.

- Install PHP dependencies
   We use the `composer` package manager to manage external PHP dependencies. To install them:

   ```bash
   composer install
   ```
- Install JavaScript Dependencies and Build Assets:
   We use the `npm` package manager to manage external JavaScript dependencies as well as using some of those dependecies to build CSS (from SCSS) and transpiling modern JavaScript to versions that can work across older and newer browsers. This is all covered by a single command which includes several steps, starting with the installation of the dependencies and ending with the complete builds of the JS and CSS assets:

   ```bash
   npm ci
   ```

#### Installing Joomla!
- Go to your localhost (port depends on how you set up your environment)
- Follow Joomla! installation instructions.

#### Installing Joomla! Patch Tester (Joomla 4.1)
With the [Patch Tester][joomla-patch-tester], you can easily apply changes from Pull Requests and test them. Alternatively, this can also be done using plain Git (and the GitHub CLI client), however we'll set this up here as this is the easiest and recommended way to get started:
- Log in to the Joomla! Backend: https://localhost/administrator
- Go to the System Menu
- Go to Extensions under the Install module
- Go to the Install from URL Tab and enter: `https://github.com/joomla-extensions/patchtester/releases/download/4.1.0/com_patchtester_4.1.0.zip`
- Click on Check & Install and follow instructions.

### Maintaining Code Style and Quality
Joomla uses Php_CodeSniffer to maintain some standards for code style and quality. We have an Joomla! Docs has an excellent guide to CodeSniffer [here][joomla-codesniffer]. An absolute must read to write good quality code that doesn't fail checks all the time.

[lamp]: https://github.com/teddysun/lamp
[podman]: https://podman.io/
[jdocs-environment]: https://docs.joomla.org/Setting_up_your_workstation_for_Joomla_development
[lamp-setup-guide]: #setting-up-an-environment-with-lamp-on-linux
[docker]: https://docker.com
[joomla-patch-tester]: https://github.com/joomla-extensions/patchtester
[jdocs-env]: https://docs.joomla.org/Special:MyLanguage/J4.x:Setting_Up_Your_Local_Environment
[scoop]: https://scoop.sh
[joomla-codesniffer]: https://docs.joomla.org/Joomla_CodeSniffer
[joomla-mariadb-official-support-thread]: https://github.com/joomla/joomla-cms/issues/14141
