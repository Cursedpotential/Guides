# Installing Docker with Docker Compose & Portainer

### Install Docker

#### Preparing the system:
##### Removing any old installations of Docker

```
sudo apt-get remove docker docker-engine docker.io containerd runc
```
##### Updates and add Docker Repo

Make sure your OS and Repos are up to date. Then we are going to add dockers repos to our system. The versions in the apt repos are old, this will ensure that we are running the newest version of the software. But first we will be installing a couple dependences.

1. Update and Install Dependencies 
```
$ sudo apt-get update

$ sudo apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
```
2. Add Dockers Official GPG key
```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```
3.  Use the following command to set up the **stable** repositorty.
```
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```
####  Install Docker

1.  Update the  `apt`  package index, and install the  _latest version_  of Docker Engine and containerd, or go to the next step to install a specific version:
    
    ```
     $ sudo apt-get update
     $ sudo apt-get install docker-ce docker-ce-cli containerd.io
    ```
#### Verify your install
This should return a result with a version number.
```
$ sudo docker --version
```
#### Post Install Setup
To create the  `docker`  group and add your user:

1.  Create the  `docker`  group.
    
    ```
    $ sudo groupadd docker
    
    ```
    
2.  Add your user to the  `docker`  group.
    
    ```
    $ sudo usermod -aG docker $USER
    
    ```
    
3.  Log out and log back in so that your group memberships are reevaluated or type the following command.
    
    ```
    $ newgrp docker 
    
    ```
    
5.  Verify that you can run  `docker`  commands without  `sudo`.
    
    ```
    $ docker run hello-world
    
    ```
    
    This command downloads a test image and runs it in a container. When the container runs, it prints a message and exits.
    
Docker is now installed!

###  Install Docker-Compose

We are using curl to install docker-compose as the packages in the repos are older. There are alternative install methods if you choose so.
1. Run this command to download docker-compose.
```
 sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```
>If you have problems installing with  `curl`, see  [Alternative Install Options](https://docs.docker.com/compose/install/#alternative-install-options)  tab above.
    
2.  Apply executable permissions to the binary:
    
    ```
    $ sudo chmod +x /usr/local/bin/docker-compose
    ```
  > **Note**: If the command  `docker-compose`  fails after installation, check your path. You can also create a symbolic link to  `/usr/bin`  or any other directory in your path.

For example:

```
$ sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```
3.  Test the installation.
    
    ```
    $ docker-compose --version
    ```
### Install Portainer

We have installed docker and verified by running the simple docker image. Now we will pull the docker portainer/portainer-ce image and run it as a container. Before running the container, create a persistent docker volume to store portainer data.
```
$ sudo docker volume create portainer_data
```
Now create the portainer container using the following command.
```
$ sudo docker run -d -p 9000:9000 -p 8000:8000 --name portainer --restart always -v /var/run/docker.sock:/var/run/docker.sock -v /srv/portainer:/data portainer/portainer-ce
```
Where:

-   -d => Run the container in detached mode
-   -p => Map the container’s port with docker host port  	
 _Port 8000 Is optional and used for edge nodes_
-   –name => Name of the container
-   -v => Volume Map

Run the following command to check the container status
```
$ sudo docker ps -a
```
You should see your new Portainer container running in the results.
Portainter is now available at http://localhost:9000.