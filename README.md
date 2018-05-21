# azure-template
azure template to create windowsserver 2016


# docker-windows-azure

## Windows Server 2016 LTS channel

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fraj2sudha1%2Fazure-template%2Fmaster%2Ftemplate.json" target="_blank">
    <img src="http://azuredeploy.net/deploybutton.png"/>
</a>


Login into Azure Vm and install git
```bash
$ choco install git -params '"/GitAndUnixToolsOnPath /WindowsTerminal"' -y
```
### Test drive

Just run it in a clean environment creating two folders on your host:

```powershell
mkdir server
mkdir client\.docker
docker run --rm `
  -e SERVER_NAME=$(hostname) `
  -e IP_ADDRESSES=127.0.0.1,192.168.254.135 `
  -v "$(pwd)\server:c:\programdata\docker" `
  -v "$(pwd)\client\.docker:c:\users\containeradministrator\.docker" raj2sudha/dockertls-windows-1709
dir server\certs.d
dir server\config
dir client\.docker
```

### Create your certs

Now create the certs and let the container

1. copy the Server certs into Docker service config folder
2. create or update the Docker service config file `daemon.json`
3. copy the Client certs into your home directory.

```powershell
mkdir $env:USERPROFILE\.docker
docker run --rm `
  -e SERVER_NAME=$(hostname) `
  -e IP_ADDRESSES=127.0.0.1,192.168.254.135 `
  -v "c:\programdata\docker:c:\programdata\docker" `
  -v "$env:USERPROFILE\.docker:c:\users\containeradministrator\.docker" raj2sudha/dockertls-windows-1709
```

Afterwards restart the Docker service in an administrator SHELL

```powershell
restart-service docker
```

Now connect to the TLS secured Docker service with

```powershell
docker --tlsverify `
  --tlscacert=$env:USERPROFILE\.docker\ca.pem `
  --tlscert=$env:USERPROFILE\.docker\cert.pem `
  --tlskey=$env:USERPROFILE\.docker\key.pem `
  -H=tcp://127.0.0.1:2376 version
```

Or just set some environment variables

```powershell
$env:DOCKER_HOST="tcp://127.0.0.1:2376"
$env:DOCKER_TLS_VERIFY="1"
docker version
```

### Create a Docker machine configuration

In addition you can create a configuration for `docker-machine`. The container then writes the TLS certs into a sub directory `machine\machines\$env:MACHINE_NAME` in your `.docker` directory and creates a `config.json` with absolute pathes using `$env:MACHHINE_HOME` environment variable. So you can build a Docker machine configuration for eg. your MacBook with Unix pathes.

```powershell
mkdir $env:USERPROFILE\.docker
docker run --rm `
  -e SERVER_NAME=$(hostname) `
  -e IP_ADDRESSES=127.0.0.1,192.168.254.135 `
  -e MACHINE_NAME=windows `
  -e MACHINE_HOME=/Users/you `
  -e MACHINE_IP=192.168.254.135 `
  -v "c:\programdata\docker:c:\programdata\docker" `
  -v "$env:USERPROFILE\.docker:c:\users\containeradministrator\.docker" raj2sudha/dockertls-windows-1709
```

Read this article : Remote Management of a Windows Docker Host
https://docs.microsoft.com/en-us/virtualization/windowscontainers/management/manage_remotehost

