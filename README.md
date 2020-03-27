# The PHP/Symfony dev instructions kit for Windows

**This repository is part of the [symfony-dev-deploy](https://github.com/RomainFallet/symfony-dev-deploy) repository.**

![Test dev env install script](https://github.com/RomainFallet/symfony-dev-windows/workflows/Test%20dev%20env%20install%20script/badge.svg)

The purpose of this repository is to provide instructions to configure a PHP/Symfony development environment on **Windows 10**.

The goal is to provide an opinionated, fully tested environment, that just work.

These instructions are also available for [macOS](https://github.com/RomainFallet/symfony-dev-macos) and [Ubuntu](https://github.com/RomainFallet/symfony-dev-ubuntu).

## Table of contents

* [Important notice](#important-notice)
* [Quickstart](#quickstart)
* [Manual configuration](#manual-configuration)
    1. [Prerequisites](#prerequisites)
    2. [Git](#git)
    3. [Symfony CLI](#symfony-cli)
    4. [PHP 7.3](#php-73)
    5. [Composer 1.9](#composer-19)
    6. [MariaDB 10.4](#mariadb-104)
    7. [NodeJS 12](#nodejs-12)
    8. [Yarn 1.21](#yarn-121)

## Important notice

Configuration script for dev environment is meant to be executed after a fresh installation of the OS.

Its purpose in not to be bullet-proof neither to handle all cases. It's  just here to get started quickly as it just executes the exact same commands listed in "manual configuration" section.

**So, if you have any trouble a non fresh-installed machine, please use "manual configuration" sections to complete your installation environment process.**

## Quickstart

[Back to top ↑](#table-of-contents)

```powershell
# Get and execute script directly
Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; Invoke-Expression ((New-Object System.Net.WebClient).DownloadString('https://raw.githubusercontent.com/RomainFallet/symfony-dev-windows/master/windows10_configure_dev_env.ps1'))
```

*See [manual instructions](#manual-configuration) for details.*

## Manual configuration

### Prerequisites

[Back to top ↑](#table-of-contents)

![chocolatey](https://user-images.githubusercontent.com/6952638/70372307-a008ed00-18dd-11ea-8288-97a9fbc7fb46.png)

On Windows 10, there is no package manager by default. We need to install the Chocolatey package manager in order to install our packages.

Open the PowerShell command prompt in administrator mode and type:

```powershell
# Install
Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))

# Reload your $PATH
$env:Path = [System.Environment]::GetEnvironmentVariable("Path","Machine") + ";" + [System.Environment]::GetEnvironmentVariable("Path","User")
```

**On Windows, after each installation, you must start a new PowerShell or reload your \$PATH** in order to use the installed packages. All command listed here must only be used inside the PowerShell in **administrator mode** (not the default command prompt).

### Git

[Back to top ↑](#table-of-contents)

![git](https://user-images.githubusercontent.com/6952638/71176962-3a1c4e00-226b-11ea-83a1-5a66bd37a68b.png)

```powershell
# Install
choco install -y git

# Reload $PATH
$env:Path = [System.Environment]::GetEnvironmentVariable("Path","Machine") + ";" + [System.Environment]::GetEnvironmentVariable("Path","User")
```

### Symfony CLI

[Back to top ↑](#table-of-contents)

![symfony](https://user-images.githubusercontent.com/6952638/71176964-3ab4e480-226b-11ea-8522-081106cbff50.png)

```powershell
# Create a new folder
New-Item -ItemType Directory -Force -Path C:\tools
New-Item -ItemType Directory -Force -Path C:\tools\symfony

# Add this folder to $PATH
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";C:\tools\symfony", "Machine")

# Determine computer architecture
IF ((Get-WmiObject -class Win32_Processor) -like '*Intel*'){$arch="386"} Else {$arch="amd64"}

# Enable TLS 1.2 (in order to connect correctly to Github)
[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12;

# Download executable from Github depending on computer architecture
(New-Object System.Net.WebClient).DownloadFile("https://github.com/symfony/cli/releases/latest/download/symfony_windows_$arch.exe", "C:\tools\symfony\symfony.exe");

# Reload $PATH
$env:Path = [System.Environment]::GetEnvironmentVariable("Path","Machine") + ";" + [System.Environment]::GetEnvironmentVariable("Path","User")
```

### PHP 7.3

[Back to top ↑](#table-of-contents)

![php](https://user-images.githubusercontent.com/6952638/70372327-bca52500-18dd-11ea-8638-7cdab7c5d6e0.png)

```powershell
# Install
choco install -y php --version=7.3.12

# Install extensions
iwr -outf C:\tools\php73\ext\php_xdebug.dll http://xdebug.org/files/php_xdebug-2.9.0-7.3-vc15-nts-x86_64.dll

# Activate extensions in php.ini
Add-Content c:\tools\php73\php.ini "extension_dir = ext"
Add-Content c:\tools\php73\php.ini "zend_extension = C:\tools\php73\ext\php_xdebug.dll"
Add-Content c:\tools\php73\php.ini "zend_extension = C:\tools\php73\ext\php_opcache.dll"
((Get-Content -path C:\tools\php73\php.ini -Raw) -replace ';extension=mbstring','extension=mbstring') | Set-Content -Path C:\tools\php73\php.ini
((Get-Content -path C:\tools\php73\php.ini -Raw) -replace ';extension=openssl','extension=openssl') | Set-Content -Path C:\tools\php73\php.ini
((Get-Content -path C:\tools\php73\php.ini -Raw) -replace ';extension=curl','extension=curl') | Set-Content -Path C:\tools\php73\php.ini
((Get-Content -path C:\tools\php73\php.ini -Raw) -replace ';extension=pdo_mysql','extension=pdo_mysql') | Set-Content -Path C:\tools\php73\php.ini
((Get-Content -path C:\tools\php73\php.ini -Raw) -replace ';extension=gd2','extension=gd2') | Set-Content -Path C:\tools\php73\php.ini
((Get-Content -path C:\tools\php73\php.ini -Raw) -replace ';extension=intl','extension=intl') | Set-Content -Path C:\tools\php73\php.ini

# Make a backup of the config file
Copy-Item -Path C:\tools\php73\php.ini -Destination C:\tools\php73\.php.ini.backup

# Update some configuration in php.ini
((Get-Content -path C:\tools\php73\php.ini -Raw) -replace 'post_max_size = 8M','post_max_size = 64M') | Set-Content -Path C:\tools\php73\php.ini
((Get-Content -path C:\tools\php73\php.ini -Raw) -replace 'upload_max_filesize = 2M','upload_max_filesize = 64M') | Set-Content -Path C:\tools\php73\php.ini
((Get-Content -path C:\tools\php73\php.ini -Raw) -replace 'memory_limit = 128M','memory_limit = -1') | Set-Content -Path C:\tools\php73\php.ini
((Get-Content -path C:\tools\php73\php.ini -Raw) -replace 'disable_functions =','disable_functions = ini_set,exec,passthru,shell_exec,system,proc_open,popen,curl_exec,curl_multi_exec,parse_ini_file,show_source') | Set-Content -Path C:\tools\php73\php.ini
((Get-Content -path C:\tools\php73\php.ini -Raw) -replace 'display_errors = Off','display_errors = On') | Set-Content -Path C:\tools\php73\php.ini
((Get-Content -path C:\tools\php73\php.ini -Raw) -replace 'display_startup_errors = Off','display_startup_errors = On') | Set-Content -Path C:\tools\php73\php.ini
((Get-Content -path C:\tools\php73\php.ini -Raw) -replace 'error_reporting = E_ALL & ~E_DEPRECATED & ~E_STRICT','error_reporting = E_ALL') | Set-Content -Path C:\tools\php73\php.ini

# Reload $PATH
$env:Path = [System.Environment]::GetEnvironmentVariable("Path","Machine") + ";" + [System.Environment]::GetEnvironmentVariable("Path","User")
```

**Installed PHP Modules:** bcmath, calendar, Core, ctype, curl, date, dom, filter, gd, hash, iconv, intl, json, libxml, mbstring, mysqlnd, openssl, pcre, PDO, pdo_mysql, Phar, readline, Reflection, session, SimpleXML, SPL, standard, tokenizer, wddx, xdebug, xml, xmlreader, xmlwriter, Zend OPcache, zip, zlib

**Installed Zend Modules:** Xdebug, Zend OPcache

### Composer 1.9

[Back to top ↑](#table-of-contents)

![composer](https://user-images.githubusercontent.com/6952638/70372308-a008ed00-18dd-11ea-9ee0-61d017dfa488.png)

```powershell
# Download installer
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"

# Create a new folder
New-Item -ItemType Directory -Force -Path C:\tools
New-Item -ItemType Directory -Force -Path C:\tools\composer

# Add this folder to $PATH
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";C:\tools\composer", "Machine")

# Install
php composer-setup.php --version=1.9.1 --install-dir=C:\tools\composer

# Remove installer
php -r "unlink('composer-setup.php');"

# Make it executable globally
New-Item -ItemType File -Path C:\tools\composer\composer.bat
Add-Content C:\tools\composer\composer.bat '@php "%~dp0composer.phar" %*'

# Reload $PATH
$env:Path = [System.Environment]::GetEnvironmentVariable("Path","Machine") + ";" + [System.Environment]::GetEnvironmentVariable("Path","User")
```

### MariaDB 10.4

[Back to top ↑](#table-of-contents)

![mariadb](https://user-images.githubusercontent.com/6952638/71176963-3a1c4e00-226b-11ea-9627-e64caabef009.png)

```powershell
# Install
choco install -y mariadb --version=10.4.8

# Reload $PATH
$env:Path = [System.Environment]::GetEnvironmentVariable("Path","Machine") + ";" + [System.Environment]::GetEnvironmentVariable("Path","User")
```

### NodeJS 12

[Back to top ↑](#table-of-contents)

![node](https://user-images.githubusercontent.com/6952638/71177167-a4cd8980-226b-11ea-9095-c96d5b96faa7.png)

```powershell
# Install
choco install -y nodejs --version=12.13.1

# Reload $PATH
$env:Path = [System.Environment]::GetEnvironmentVariable("Path","Machine") + ";" + [System.Environment]::GetEnvironmentVariable("Path","User")
```

### Yarn 1.21

[Back to top ↑](#table-of-contents)

![yarn](https://user-images.githubusercontent.com/6952638/70372314-a13a1a00-18dd-11ea-9cdb-7b976c2beab8.png)

```powershell
# Install
choco install -y yarn --version=1.21.1

# Reload $PATH
$env:Path = [System.Environment]::GetEnvironmentVariable("Path","Machine") + ";" + [System.Environment]::GetEnvironmentVariable("Path","User")
```
