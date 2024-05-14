# Subject

we want to:

1. pull `ubuntu` image.
2. run an interactive container named `base-ubuntu` from it.
3. install NodeJS in this `base-ubuntu` container.
4. create an image named `node-ubuntu` from this running container.
5. run container named `node-ubuntu-c1` from image `node-ubuntu`.
6. test container `node-ubuntu-c1`. 



# Pull `Ubuntu` Image

run following command:

```powershell
docker pull ubuntu
```





***

# Run And Setup Base Container

execute following command:

```powershell
docker run --name=base-ubuntu -ti ubuntu
```



after above command executed, you enter into `base-ubuntu` Linux container. so run the following commands respectively:

```powershell
apt update
apt install -y nodejs			
```



to test the `nodejs` successful installation, execute following commands:

```powershell
node --version
node -s 'console.log("Test from base-ubuntu")'
```





***

# Create Image From Container

execute following command to create an image named `node-ubuntu` from container `base-ubuntu`:

```powershell
docker container commit -m "Add Node" base-ubuntu node-ubuntu
```



after above command executed and image named `node-ubuntu` is created with one more layer with comment `Add Note`.



to view the layers of the image `node-ubuntu` execute following command:

```powershell
docker image history node-ubuntu
```



***

# Run and Test node container

execute following command to run a new container named `node-ubuntu-c1` from image `node-ubuntu`:

```powershell
docker run --name=node-ubuntu-c1 -ti node-ubuntu	
```



to test that our new container works as expected run the following commands:

```
node --version
node -s 'console.log("Test From node-ubuntu")'
```

