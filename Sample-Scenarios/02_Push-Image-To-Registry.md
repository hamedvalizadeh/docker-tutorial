# Subject

we want to:

1. create a public repository named `test` in docker hub panel. 
2. login to our docker hub account in terminal.
3. clone a simple node app from a public git repository.
4. create an image named `docker-hello` from it in our local machine.
5. tag our newly created image as `1.0`.
6. push it to the repository `test` in docker hub registry. 



# Create Repository

login to `https://hub.docker.com` and click on `Repositories` menu, in the opened page click the button `Create repository`, in the `Create` page select `Namespace` and enter a name for your repository, and select `Public` as `Visiblity` property of your repository and finally click `Create` button. that's it.



***



# Login

run the following command:

- docker login -u [your user name]

you will be prompted to enter password. 



***



# Clone Sample Node App

create a directory and cd to it and run the following command:

```git
git clone https://github.com/dockersamples/helloworld-demo-node
```



cd to `helloworld-demo-node`

run the following command:

```powershell
docker build -t docker-hello .
```



run the following command to see the list of images. you should see an image named `docker-hello`:

```
docker images
```



to test if the image is correct, run the following command and the navigate to `http://localhost:8080`:

```
docker run -d -p 8080:8080 docker-hello
```



# Tag Image

tagging allows you to label and version your image. to tag `docker-hello`, run the following command:

```powershell
docker tag <local image name> <your username>/<repository name>:<version>
```



imaging your username is `dockeruser`, so the complete command would be as follow:

```powershell
docker tag docker-hello dockeruser/test:1.0	
```

 

***



# Push

run the following command:

```powershell
docker push <user name>/<repository>:<version>

docker push dockerusername/test:1.0
```



now navigate to your docker hub panel and open `test` repository, under tags you could see your image have been pushed and is listed.
