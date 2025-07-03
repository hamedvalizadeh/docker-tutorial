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



now to be able to pull docker images, it is recommended to change docker image hub repository to somewhere which is not banned. one of the possible mirror repository available among many possible ones could be for `arvancload` (based of link `https://www.arvancloud.ir/fa/dev/docker`). so do the following:

- first change registry in `daemon.json`, by executing following command:

- ```
  sudo bash -c 'cat > /etc/docker/daemon.json <<EOF
  {
    "insecure-registries" : ["https://docker.arvancloud.ir"],
    "registry-mirrors": ["https://docker.arvancloud.ir"]
  }
  EOF'
  ```

- then execute following command for changes to be applied:

- ```
  docker logout
  sudo systemctl restart docker
  ```

  

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
