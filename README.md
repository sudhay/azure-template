# azure-template
azure template to create windowsserver 2016


# docker-windows-azure

## Windows Server 2016 LTS channel

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fraj2sudha1%2Fazure-template%2Fmaster%2Ftemplate.json" target="_blank">
    <img src="http://azuredeploy.net/deploybutton.png"/>
</a>

## Windows Server 1709 LTS channel
<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fraj2sudha1%2Fazure-template%2Fmaster%2Ftemplate-1709.json" target="_blank">    <img src="http://azuredeploy.net/deploybutton.png"/></a>


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


Below steps are taken from stephan scherer github io article
https://stefanscherer.github.io/protecting-a-windows-2016-docker-engine-with-tls/
### Create Certificates in Azure VM
PS C:\Users\sy> docker run --rm -e SERVER_NAME=#azureVMhostName#.westeurope.cloudapp.azure.com -e IP_ADDRESSES=127.0.0.1,#azureVMstaticIP# -v C:\ProgramData\docker:c:\programData\docker -v $env:USERPROFILE\.docker:C:\Users\ContainerAdministrator\.docker raj2sudha/dockertls-windows-1709    <br/> Directory: C:  <br/>Mode                LastWriteTime         Length Name  <br/>----                -------------         ------ ----  <br/>d-----        5/21/2018  12:38 PM                DockerSSLCARoot  <br/>=== Generating CA private password  <br/>=== Writing out private key password  <br/>=== Generating CA private keyGenerating RSA private key, 4096 bit long modulus ....................................++............................++e is 65537 (0x10001)  <br/>=== Generating CA public key === Reading in CA Private Key Password  <br/>=== Generating Server private keyGenerating RSA private key, 4096 bit long modulus ..............................................................................++..........................................................................++e is 65537 (0x10001)  <br/>=== Generating Server signing request  <br/>=== Signing Server requestsubjectAltName = IP:127.0.0.1,IP:#azureVMStaticIP#,DNS.1:#azureVMhostName#.westeurope.cloudapp.azure.comSignature oksubject=/CN=#azureVMhostName#.westeurope.cloudapp.azure.comGetting CA Private Key  <br/>=== Generating Client keyGenerating RSA private key, 4096 bit long modulus .........................++...................................................................................++e is 65537 (0x10001)  <br/>=== Generating Client signing request  <br/>=== Signing Client signing requestSignature oksubject=/CN=clientGetting CA Private Key  <br/>=== Copying Server certificates to C:\ProgramData\docker\certs.d  <br/>=== Copying Client certificates to C:\Users\ContainerAdministrator.docker  <br/>=== Creating / Updating C:\ProgramData\docker\config\daemon.json  <br/>=== FinishedNow restart Docker service with the following command: <br/>restart-service docker
Azure VM : docker version  <br/>PS C:\Users\sy.docker> docker version  <br/>Client:   <br/>Version:      17.10.0-ee-preview-3   <br/>API version:  1.33   <br/>Go version:   go1.8.4   <br/>Git commit:   1649af8   <br/>Built:        Fri Oct  6 17:52:28 2017   <br/>OS/Arch:      windows/amd64 
<br/>Server:  <br/>Version:      17.10.0-ee-preview-3 <br/>API version:  1.34 (minimum version 1.24) <br/>Go version:   go1.8.4  <br/>Git commit:   b8571fd  <br/>Built:        Fri Oct  6 18:01:48 2017  <br/>OS/Arch:      windows/amd64  <br/>Experimental: false


### Copy Certificates from Azure VM to local PC

Azure VM Certificate Path : $env:USERPROFILE\.docker
<br/>local PC Certificate Path : $env:USERPROFILE\.docker

Then Update the Environment Variables in local PC
<br/>PS C:\Users\sy> ls env:DOCKER_*
<br/>Name                           Value
<br/>----                           -----
<br/>DOCKER_TOOLBOX_INSTALL_PATH    C:\Program Files\Docker Toolbox
<br/>DOCKER_HOST                    tcp://#azureVMhostName#.westeurope.cloudapp.azure.com:2376
<br/>DOCKER_TLS_VERIFY              1
<br/>DOCKER_MACHINE_NAME            default
<br/>DOCKER_CERT_PATH               C:\Users\sy\.docker
    
<br/>PS C:\Users\sy> docker version
<br/>Client:         18.03.0-ce 
<br/>API version:   go1.9.4owngraded from 1.37) 
<br/>Git comFri Mar 23 08:31:36 2018 
<br/>OS/Arch:       falsews/amd64 
<br/>Orchestrator:  swarm

<br/>Server: 
<br/>Engine:        17.10.0-ee-preview-3 
<br/>API version:  go1.8.4inimum version 1.24)  
<br/>Git commit:   Fri Oct  6 18:01:48 2017  
<br/>OS/Arch:      falsews/amd64


<br/>=============================================
<br/>After this we have to copy the azure VM from ~/.docker/machine/machines/#azureVMhostName#
<br/>to local VM path ~/.docker/machine/machines/

<br/>For me both azure VM and local VM had the same Users (C:\Users\sy)

<br/>For you if they are different.
<br/>Go to local VM path : ~/.docker/machine/machines/#azureVMhostName# 
<br/>Open Config.json and at end of the files Cert paths should be updated to your local user.



 "AuthOptions": {            
 "CertDir": "/Users/<localUser>/.docker/machine/machines/#azureVMhostName#",            
 "CaCertPath": "/Users/<localUser>/.docker/ca.pem",            
 "CaPrivateKeyPath": "/Users/<localUser>/.docker/machine/machines/#azureVMhostName#/ca-key.pem",            
 "CaCertRemotePath": "",            
 "ServerCertPath": "/Users/<localUser>/.docker/machine/machines/#azureVMhostName#/server.pem",           
 "ServerKeyPath": "/Users/<localUser>/.docker/machine/machines/#azureVMhostName#/server-key.pem",           
 "ClientKeyPath": "/Users/<localUser>/.docker/key.pem",            
 "ServerCertRemotePath": "",           
 "ServerKeyRemotePath": "",           
 "ClientCertPath": "/Users/<localUser>/.docker/cert.pem",           
 "ServerCertSANs": [],            
 "StorePath": "/Users/<localUser>/.docker/machine/machines/#azureVMhostName#"        
 }
