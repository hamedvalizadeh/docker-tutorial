# What Is It?

Is an open platform for developing, shipping, and running applications without any dependency to the host environment. it separates your application from host infrastructure, so cause software delivery to be mush more quicker, easier, safer, and stable. 



***

# Docker Structure

docker uses client-server architecture. there is an app that is running in client which communicates with a running app in the server with REST API. both of these apps could be on the same machines or be separated in local and host.



### Docker Daemon (dockerd)

this is the app which is running in host machine and is responsible of managing and creating images, containers, networks, and volumes. this app listens to the client API requests and execute the appropriate command in docker engine. 



### Docker Client (docker)

is an app that provides `CLI` for users to communicate with Daemon. this app sends the user entered commands to the `dockerd` to execute them in docker engine. this app uses Docker API to communicate with `dockerd`.



### Docker Desktop

this is an application installable on Mas, Linux, ore Windows, that consists of `dockecr client`, `docker daemon`, `docker compose`, `docker content trust`, `kubernetes`, and `credential helper`.



***

# Install Docker

as some distro maintainers include some unofficial packages in their Linux distribution, it is recommended to uninstall those packages before installing the docker. so run the following command to uninstall them:

- dpkg -r `docker.io`
- dpkg -r `docker-compose`
- dpkg -r `docker-doc`
- dpkg -r `podman-docker` 



If you have installed the `containerd` or `runc` previously, uninstall them to avoid conflicts with the versions bundled with Docker Engine:

- dpkg -r `containerd`
- dpkg -r `runc` 



before installing docker in new machine, you need to set up docker `apt` repository. so run the following commands:

1. apt-get update

2. apt-get install ca-certificates curl

3. install -m 0755 -d /etc/apt/keyrings

4. curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc

5. chmod a+r /etc/apt/keyrings/docker.asc

6. ```
   echo \
     "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/debian \
     $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
     sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
   sudo apt-get update 
   ```



now everything is ready to install the docker. so run the following command:

- apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin



after installing docker, to test it is installed correctly, run the following command:

- docker run hello-world
  - When the container runs, it prints a confirmation message and exits.



***

# Install Docker-Desktop

if you are doing the installation in Virtual Machine gust OS, you should enable Virtualization from following path in `Virtual Box` program:

- settings >> System Menu >> Processor Tab >> enable Check-box `Enable Nested VT-x/AMD-V`



download docker-desktop package from following URL:

- ```
  https://desktop.docker.com/linux/main/amd64/139021/docker-desktop-4.28.0-amd64.deb?utm_source=docker&utm_medium=webreferral&utm_campaign=docs-driven-download-linux-amd64&_gl=1*zrj649*_ga*MTA1MTE5NzY4My4xNzEyMjk2MTcy*_ga_XJWPQMJYHQ*MTcxMjQ4MjM3OC42LjEuMTcxMjQ4NjY1Ny4zMi4wLjA.
  ```



in terminal environment navigate to the downloaded package folder and execute following commands:

- apt install gnome-terminal

- apt-get update

- apt-get install ./docker-desktop-<version>-<arch>.deb

- after installation finished, following error message is printed, you can ignore it:

  -  

    ```
    Download is performed unsandboxed as root, as file '/home/user/Downloads/docker-desktop.deb' couldn't be accessed by user '_apt'. - pkgAcquire::Run (13: Permission denied)
    ```



now to launch `docker desktop` search it in `Appications` menu or run the following command:

- systemctl --user start docker-desktop



# Docker Context

each user has its own context that when an image is build under that user account, the built image is stored in that context. to see the docker context and some other info, execute the following command:

- docker info



it is possible to transfer images between different contexts. to do so, execute the following command:

- docker -c [source context] save [image name] | docker -c [destination context] load



in order to clarify the  above command, imaging we have a non-root user that is running docker in a context named (( desktop-linux )) and created an image named (( wtd )) in it, and want to transfer it to the context name (( default )) of the root user. the command would be as follow:

- docker -c desktop-linux save wtd | sudo docker -c default load 



# Registry

to explain it as simple as possible, this is a centralized location in which docker images are stored and can be shared to other developers.

a registry could be public or private.

a public registry is a hosted location in internet that could be accessed by every-one in the the internet. some of famous and reliable public image repositories are as follow:

- docker hub: https://hub.docker.com
- Amazon Elastic Container: https://aws.amazon.com/ecr
- Azure Container: https://azure.microsoft.com/en-in/products/container-registry
- Google Artifact: https://cload.google.com/arifact-registry



### Repository

is a collection of related images within a registry. in a simple term, it is some how like a sub folder in registry.
