# Subject

we want to:

1. create a simple API with NodeJS and Express that is running on port 5000.
2. dockerize the API that responses to port 5001 in the host.
3. dockerize a reverse proxy with Nginx, to send the request to the local host with port 8080 and route (( /sample )) to the hosted API in port 5001.



# Create API

run the following commands to initiate the node app config file and install Express:

- mkdir simple_node
- cd simple_node
- npm init -y
- npm install express

now create a file file named (( `index.js` )) and paste the following content to it:

```javascript
const express = require('express'); 
const app = express(); 
  
app.use(express.urlencoded({ extended: true }));
app.use(express.json());
  
app.get('/',  
   (req, res) => res.send('Dockerizing Node Application'));
  
app.get('/sample',  
   (req, res) => res.send('sample Dockerizing Node Application');
  
app.listen(5000,  
   () => console.log('Server is running at port: 5000'));
```



# Create Docker Image of API

cd to the folder (( `simple_node `)).

create a file named (( `Dockerfile.dev` )) and paste following command to it:

```dockerfile
# Fetching the minified node image on apline linux
FROM node:slim

# Declaring env
ENV NODE_ENV development

# Setting up the work directory
WORKDIR /simple_node

# Copying all the files in our project
COPY . .

# Installing dependencies
RUN npm install

# Starting our application
CMD [ "node", "index.js" ]

# Exposing server port
EXPOSE 5000
```



create a file named (( `.dockerignore` )) and paste the following lines to it, to exclude unnecessary files to be included in created docker image: 

```dockerfile
node_modules
npm-debug.log
build
.git
*.md
.gitignore
```



run the following command to build the image:

```
commandformat:
 - docker build -f Dockerfile.dev -t [name:tag] .
 
example command:
 - docker build -f Dockerfile.dev -t simple_node_api:1.0 .
```



- **-f:** Path to the docker file

- **-t:** Name and tag for the image

- **. :** The context for the build process



after running this command an image named (( `simple_node_api` )) with tag (( `1.0` )) should be created and be added to your images list. to check it run the following command:

- docker image ls



# Containerize Node API

to run the app in a docker container run the following command:

```
command format:

docker run -d -it –-rm -p [host_port]:[container_port] –name [container_name] [image_id/image_tag]
 
command sample:

docker run -d -it –-rm -p 5001:5000 –name sn_api simple_node_api:1.0
```



to test the result open a browser and request (( `http:/localhost:5001` )), you should se the message (( `Dockerizing Node Application` )).



**Caution:** run the command (( `ifconfig ` )) and in the result list look for a network that its name contains (( `docker` )), and memorize it as (( `node API docker IP` )).



# Containerize Nginx as Reverse Proxy

run the following command to pull latest version of nginx image from docker hub:

- docker pull nginx

now execute the following command to run a container named (( `proxy-api` )) from nginx image on port `8080`of hosting machine:

- docker run -d --name proxy-api -p 8080:80 nginx



nginx by default is configured as a file server; to configure it as a reverse proxy  we should change its default config file. to do so, execute following commands:

- create a folder named (( `docker_conf` ))

- cd docker_conf

- create a file named (( `default.conf` )) and paste following script to it:

  - ```nginx
    server {
        listen       80;
        listen  [::]:80;
        server_name  localhost;
    
        location / {
            root   /usr/share/nginx/html;
            index  index.html index.htm;
        }
    
        location /sample {
            proxy_pass http://[node API docker IP]:5001;
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection 'upgrade';
            proxy_set_header Host $host;
            proxy_cache_bypass $http_upgrade;
        }
    
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   /usr/share/nginx/html;
        }
    }
    ```

    

now we need to copy above config file inside docker container named (( `proxy-api` )) and replace its `default.conf` file with new one. so execute following command:

- docker cp  /docker_conf/default.conf  proxy-api:/etc/nginx/conf.d/default.conf



after it check the correctness of our new replace config file in container (( `proxy-api` )) by running following command. this command should print a message containing (( `successful` )):

- docker exec proxy-api -t

finally to reload the config file in containerized nginx, execute following command:

- docker exec proxy-api -s reload



# Test

to test the result open a browser and request to the following URLs:

- http://localhost:8080
  - you should see the default page of nginx server.
- http://localhost:8080/sample
  - you should se the message (( sample Dockerizing Node Application ))
