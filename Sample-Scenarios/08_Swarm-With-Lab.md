# Subject

we want to:

1. login to play with docker (PWD).
1. setup our servers.
1. setup SWARM environment.
1. run a simple container from image `recotik/pwd:1.0.0` on one node.
1. scale up our container to all nodes.



# PWD

it stands for `play with docker`; this is a web-site that is held and managed by docker team to make it possible to learn docker in practice without need to have Linux OS on our local machines.

it has some amazing capability, that one is possibility for the user to create multiple servers, and work with them to learn and test docker swarm in practice.

OS of servers in PWD are `alpine` version of Linux.

to begin our scenario, go to site `https://labs.play-with-docker.com/`, click on `Login` button and after being authenticated successfully, click on `Start` button. by result you will be redirected to a web page that is your laboratory to work in; it is called `session` in `PWD`.



***

# Setup Servers

for our scenario we need three servers (Linux server). to do so click on button `+ ADD NEW INSTANCE` in your `session`, this will create a server for you and list it in left pane. repeat this action for 2 more times to have 3 servers at all. each server in this pane is distinguished by it's IP from others; say we have servers with following IPs:

- 192.168.0.28
- 192.168.0.27
- 192.168.0.26



***

# Setup SWARM Environment

click on server `192.168.0.28` from left pane, and execute the following command to activate swarm capability on this server.

```powershell
docker swarm init --adverise-addr $(hostname -i)
```



after running above command, the current server will be a manager node in our swarm environment. to test it execute following command to see available nodes in our swarm environment:

```powershell
docker node ls
```

 

you will see a list consisting of a manager  (Leading) node. this command is just executable in swarm enabled nodes, that are manager nodes.



to join another server to our swarm environment, we need a token. this token is achieved from a manager node. the structure of the join command is as follow:

```powershell
docker swarm join --token [token value] [manager node IP]:2377
```

 

we can join a new node as manger, or as worker. the structure of join commands to add new node as manager or as a worker are  the same, but the value of the token is different.

to gain the token to add a new node as worker, run the following command in one of a docker swarm enabled manager node in our swarm environment, that in our scenario it is server `192.168.0.28`:

```powershell
docker swarm join-token -q worker
```

  

to gain the token to add a new node as manager, run the following command in one of a docker swarm enabled manager node in our swarm environment, that in our scenario it is server `192.168.0.28`:

```powershell
docker swarm join-token -q manager
```



as we want 2 other servers (in our scenario they are as `192.168.0.27` and `192.168.0.26`) to be workers, so copy the worker token (say it is `SWMTKN-1-4ehd6rsgtus8ra1oa3p4awrbn73hkix9j327r4rriv6y0opvhy-7n3mkre32rbazycz3yfvlrn4s`) and run the following command in every 2 other servers:

```powershell
docker swarm join --token SWMTKN-1-4ehd6rsgtus8ra1oa3p4awrbn73hkix9j327r4rriv6y0opvhy-7n3mkre32rbazycz3yfvlrn4s 192.168.0.28:2377
```



**HINT:** to copy in PWD select your content by mouse and press `ctlr + insert`; to paste you have 2 options, either press `shift + insert`, or press `shift + ctrl + v`.



**HINT:** you can run the following command in a manager node, and copy the result and run that in a server that you want to join it as a worker to your swarm environment:

```powershell
echo "docker swarm join --token"  "$(docker swarm join-token -q worker)" "$(hostname -i):2377"
```



now run the following command in manager node `192.168.0.28` again. you will see three nodes in the list; one is manager that is server `192.168.0.28`, and 2 others are workers that are `192.168.0.27` and `192.168.0.16`.



# Run Container 

in swarm environment we have servers and usually our goal is to load balance requests to a containers of the same image (all are same exact copy of an image and doing exactly same task) between these servers. so in first step, to make it possible for swarm to run the container in each server by its choice, it is needed that the image of the container be available in each server locally. as we want to run a container from the image `recotik/pwd:1.0.0`, so we should pull it in each server separately; to do so, execute the following command one time in each server:

```powershell
docker pull recotik/pwd:1.0.0
```



execute the following command in the manager node `192.168.0.28`:

```powershell
docker service create -d --name s-api --replicas 1 -p 3001:3000 recotik/pwd:1.0.0
```



this command will run one container from an image tagged `recotik/pwd:1.0.0`, as a service named `s-api` on a node in our swarm environment. to see in which node the service is running, execute the following command in manager:

```powershell
docker service ps
```



in our example the container is running in server `192.168.0.28` which is manager. choosing the server to run the container in it by docker swarm, is not related to which server the `docker service create` command is executed on.



as our service `s-api` is started with one container running on server `192.168.0.28`, you can query containers list by executing command `docker ps` in this server to see that one container that it's name starts with `s-api`  is running in it.



to test the result, click on the button `OPEN PORT` in web page related to the server `192.168.0.28`, in the opened pop up enter 3001 and click om `ok` button; you will be navigated to a new web browser tab that will show you a `json` string as response, it should be `{"status":"ok"}`. at the end of page address type `/names` and press enter, you will get another response that is in `json` format and is a list of strings.



***

# Scale UP

by here we have set up 3 servers, but deployed one instance of our image in one of the; to maximize our usage from our initiated infrastructure, it is logical to have one instance of the image in each of these servers. to do so, in manager server `192.168.0.28` run the following command:

```powershell
docker service scale s-pai=3
```

 

this will cause our service `s-api` run 3 containers of our image in its swarm environment available nodes.



we have 3 servers and scale it to have 3 container, so docker swarm most likely will run 1 container in each server; but it is possible to run 2 containers in one server, and nothing in 1 remaining server.



if you scale to more than 3, and after that execute the command `docker service ps s-api`, you will see that some nodes have more instances than others.  