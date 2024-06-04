# What Is It?

Is an open platform for developing, shipping, and running applications without any dependency to the host environment. it separates your application from host infrastructure, so cause software delivery to be mush more quicker, easier, safer, and stable. 



***

# Docker Structure

docker uses client-server architecture. there is an app that is running in client which communicates with a running app in the server with REST API. both of these apps could be on the same machines or be separated in local and host.



### Docker Daemon (dockerd)

this is the app which is running in host machine and is responsible of managing and creating images, containers, networks, and volumes. this app listens to the client API requests and execute the appropriate command in docker engine. there is a config file for it, that reside in `~/.docker/daemon.json` or `/etc/docker/deamon.json`.



### Docker Client (docker)

is an app that provides `CLI` for users to communicate with Daemon. this app sends the user entered commands to the `dockerd` to execute them in docker engine. this app uses Docker API to communicate with `dockerd`.



### Docker Desktop

this is an application installable on Mas, Linux, ore Windows, that consists of `dockecr client`, `docker daemon`, `docker compose`, `docker content trust`, `kubernetes`, and `credential helper`.



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
