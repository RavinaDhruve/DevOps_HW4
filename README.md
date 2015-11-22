# CSC 791 DevOps, Fall 2015

## Homework Assignment #4

**Name:** *Ravina Dhruve*
**Unity ID:** *rrdhruve*
___


ADVANCED DOCKER
=========================

PRE-REQUISITES:

- Using DigitalOcean to install use droplets to deploy docker containers.
- Install docker-io on the droplet
- Using upgraded docker version (1.9)
- Installing docker-compose


### TASK 1: FILE IO

On a droplet: 

+ Image creation using Dockerfile
```
sudo docker build -t socat_container .
```

+ We have two docker containers:

	- Server container : It runs a command script that outputs current date and time to a file (output.txt)
	```
	sudo docker run -td --name container_1 socat_container
	sudo docker exec -it container_1 bash
	sh server_script.sh > output.txt
	socat tcp-l:9001,reuseaddr,fork system:'cat /output.txt',nofork
	```

	- Client container: It is linked with the container and fetches the output.txt using curl command from
					    Server container listening on 9001 (using socat connection)
	```
	sudo docker run -td --link container_1:server --name container_2 socat_container
	sudo docker exec -it container_2 bash
	sudo apt-get install curl
	curl server:9001 > result.txt
	cat result.txt
	```

Note: The containers are created using Dockerfile, included as a part of the repo in Task1/ folder.



### TASK 2: AMBASSADOR PATTERN

Note: Make sure docker-compose is installed on the droplet hosts.

To install docker-compose:
```
curl -L https://github.com/docker/compose/releases/download/1.5.1/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
```

+ There are two droplet machines – Host 1 and Host 2 to isolate docker hosts.

+ Using Docker compose, the following 4 containers are spun up:
	- Host 1 (Server):
		- redis : It is the redis server.
		- redis_ambassador_server : This container is a linked container to redis.
	```
	docker-compose up
	```

	- Host 2 (Client):
		- redis_ambassador_client : This container talks to Host1 (Server) over a TCP connection
              						 to perform /set and /get requests from client_cli container 
              						 and redirect it to Host1 redis server.
    	- client_cli container : This is a linked container to redis_ambassador_client which makes
								  the /set and /get requests.
	```
	docker-compose up
	sudo docker run -it --link redis_ambassador_client:redis --name client_cli relateiq/redis-cli
	```

+ The /set and /get requests fired from client_cli are getting serviced using the redis_server store
  on Host1 (Server).

Note: The containers with their linking and other variables are configured and created using
      docker-compose.yml, included as a part of the repo in Task2/ folder.(in host1/ and host2/)


	
### TASK 3: DOCKER DEPLOY

Note: The following command needs to be executed in the beginning to create a local registry:
```
docker run -d -p 5000:5000 --restart=always --name registry registry:2
```

+ An App commits and pushes the code to “blue” and “green” slice created. (from the Deployment
  and Advanced Docker workshop)

+ On a push to blue/green slice, following tasks are executed:
	- A new docker image is created using Dockerfile from repo

	- This image is tagged and pushed to local registry (pre-created using a registry container
	  which holds all the images)

	- The image is then pulled from the registry.

	- A new docker container is deployed using the image, with the
	  app.js web server listening for requests.

	- The hooks commands pull the image, stop any existing containers
	  and restart containers with the new code.


Link to App: https://github.com/CSC-DevOps/App

Note:
+ The containers are configured and created using image created using Dockerfile
  included as a part of the repo in Task3/ folder.

+ The post-receive hooks of both blue and green slice are included as a part of the
  repo in Task3/ folder.(in blue_slice/ and green_slice/)
  
+ For the purposes of demo as shown in the screencast, the server on blue slice listens on 
  port 8080 and that on the green slice listens on port 8081 externally.



### SCREENCAST LINKS:

COMPONENT FILE IO : https://youtu.be/jYK9xTysU-c

COMPONENT AMBASSADOR PATTERN : https://youtu.be/ek3VOKXsnn0

COMPONENT DOCKER DEPLOY : https://youtu.be/WNf06xA4jKI

Tool used: QuickTime player
___


**File Description:**

+ README.md - this current file.
+ Task1/ - Folder including Dockerfile for creating containers in Task 1.
+ Task2/ - Folder including docker-compose.yml for creating linked containers in Task 2.
+ Task3/ - Folder including Dockerfile and Post-receive hooks for creating containers in Task 3.
