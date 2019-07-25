# Docker_And_K8s

# Docker
  * it creates a linux VM and runs on the OS
  * Commands - Section 2, L-13
    * creating and running an container from the image
      - docker run <image-name>
        * docker run <image-name> = docker create <image-name> + docker start <container-ID>
    * Override command during start up or run an command after startup of container after it is created
      - docker run <image-name> command!   
    * To list all the running containers on machine
      - docker ps
    * To List all the container that had been run
      - docker ps --all || docker ps
    * To delete all the stopped containers, it will take up space
      - docker system prune  
    * To stop a running container, started using create and start
      - docker stop <container-id> || docker kill <container-id>  
        * stop => sends a "SIGTERM" signal to the process to shut down by giving it some time to do some clean up, like saving a  file, emit a message etc.
                  if in 10 sec's it does not stop process will fall back to "kill"
        * kill => sends a "SIGKILL" signal to the process to tell it shut down right now
    * Container Lifecycle
      - docker run <image-name> = docker create <image-name> + docker start <container-ID>   
      * need to use docker start -a <container-ID>   => -a watches the container for o/p and prints it to console
      * if we forget to use "-a" we can use => docker logs <container-id>
      * docker logs -f <C_ID>
      
    * Executing commands in running Containers
      - docker exec -it <container-id> <command>
        * it allows us to provide input to the container
        * example:
          * first run : docker run redis
          * to access cli : docker exec -it xxxx redis-cli
      * Purpose of -it flag (-i => to attch the STDIN to the terminal, -t => to show pretty text output)
        * Each process or container which is running has three standard channels
          * STDIN => -it
          * STDOUT  => -a flag or logs
          * STDERR  => similar to stdout, uses STDOUT to throw error    
    
    * Get Shell or Terminal Access to the required container
      - docker exec -it <container-id> sh     
      * different command process bash, powershell, zsh, sh
      
    * Start Container with Shell terminal
      - docker run -it <image-name> sh
      
    * when starting two different containers of same image, the containers have their own space for files other things, so when we create a 
      a file in container1 of a image, it won't be reflected on the other container2    
      
  * Building Custom Images Through Docker Server - Section -3
    * Creating Docker Images - 27
      1) we need to create a docker configuration file which is a plain text, this configuration defines how our container behaves.  
      2) then we pass it to the docker client(docker cli)  
      3) the docker client gives the text file it to the docker server and this does the heavy lifting, to build a usable image
      
    * Flow
      * we need to specify a base image -> run some commands to install additional programs -> specify a command to run on container setup 
    
    * Docker File Tear Down
      * new to create a docker file with name "Dockerfile" without any extension
      * Tear Down
        
        instructions telling  |   Argument to the      
         what Docker Server   |     instructions
            to do             |
            
            FROM                  alpine
            RUN                   apk add --update redis  
            CMD                   ["redis-server"]
            
      * From location of the file where it is stored, run
        - docker build .
        - docker run <image-id> => image-id => o/p by running previous command      
        
      * Tagging Image (instead of using image id to run everytime we can tag like redis-server or helloworld)
        - docker build -t <tagname> <specify the directory of file/folder to use for build>
          example: docker build -t stephengrider/redis:latest  .
          
          - rajdiv23/redis:latest  
            yourdockerid / repo or project name(anything) : version
            
          * TO run
            - docker run  rajdiv23/redis 
            
  * Making Real Projects with Docker - Section 4
    * To copy files from local file system(Hard drive) to the container's file system 
      - COPY <path to folder to copy from on *your machine* relative to build text> <Place to copy stuff to inside *the container*>    
        - COPY ./ ./    
        
    * To map local machine port to the container port - Port Forwarding
      - docker run -p <port of local machine>:<port inside container> <image-id>     
        - docker run -p 8080:8080 rajdiv23/simpleweb => localhost:8080/
        - docker run -p 5000:8080 rajdiv23/simpleweb => localhost:5000/
        
    * Specifying the directory to put all copy all the files into
      - WORKDIR /usr/app => any following command will be executed relative to this path in the container 
      * if folder not present it will be automatically created for us
      
    * Rebuilds
      * making sure only specific files are built
      
  * Intro to Docker Compose
    * used to issue multiple commands more quickly
    * Command 
      - docker-compose       
    * When using docker compose and creating multiple containers, they would be created on the same network and communication between those containers
      would become easy. To reffer the service to be used on the same container, for example a redis-server, we need to mention the name provided in the
      YAML file to the host property of the container that want's to use it. Section 5, 53
    * Commands to run:
      - docker run <myimage> => docker-compose up
      
      * Rebuild
        - docker build .          | =>  docker-compose up --build
          docker run <myimage>    | =>   
    
      * Launch in background
        - docker-compose up -d
        
      * stop containers
        - docker-compose down        
        
      * Deal with containers that crash
        * Automatic Container restarts 
          * restart policies
            * no, always, on-failure, unless-stopped
      
      * Container Status with Docker Compose
        * need to run from folder which has "docker-compose.yml"
        -  docker-compose ps      
      
  * Build Work Flow 
    * Run a docker file of custom type
      * docker build -f Dockerfile.dev . 
    
    * Docker Volumes section 6, 69
      * make changes take immediate effect      
      * instead of copying all the files, we are going to map the file and folders in our local machine to the one in inside the container. 
      * Command
        - docker run -p 3000:3000 -v /app/node_modules -v $(pwd):/app <imageid>
                                 | put a bookmark on  | Map the pwd
                                   the node_modules   | into the /app 
                                   folder             | folder
                                   
          * -v $(pwd):/app => means map the current pwd on the local machine to the /app folder in the container                         
      
      * Run Tests => section 6, 74
        * docker build -f Dockerfile.dev .
        * docker run -it 18fb7187a2b6 npm run test => override the default command  
        
      * Reflect changes made to test files without adding volumes
        * docker-compose up
        * in another terminal, run (use docker ps)
          - docker exec -it <imageid> npm run test 
          
        * Better Solution
          *  Adding another service    
          * docker-compose.yml
            - version: '3'
              services:
                web:
                  build: 
                    context: .
                    dockerfile: Dockerfile.dev
                  ports:
                    - "3000:3000"
                  volumes:
                    - /app/node_modules
                    - .:/app  
                    
          * run, docker-compose up --build      
          
      * Multi Step Build Process
        *  Section 6, 79-80-81
        
  * Set up Docker, elastic Beankstalk  Section 7, 89  
  
  * Multi Container Application Section-9
  
  * Setting env variable inside a container, section 9, 117
  
  * Production multi container deployment
  
  
  Delete Images forcefully
    : docker rmi $(docker images -q) -f
    
    * Container : 
      * docker rm <CONTAINER ID>
    
    * https://stackoverflow.com/questions/38118791/can-t-delete-docker-image-with-dependent-child-images
    * https://www.digitalocean.com/community/tutorials/how-to-remove-docker-images-containers-and-volumes   
    
  * Inspecting docker images 
    docker image inspect mongo     
    
  Cleaning up containers Section 3, 23 JT
    * kill all running containers
     - docker kill $(docker ps -q)
    * delete all stopped docker Containers
     - docker rm $(docker ps -a -q)
  
 * Cloning Private Repos:
   https://stackoverflow.com/questions/23391839/clone-private-git-repo-with-dockerfile
   https://stackoverflow.com/questions/18136389/using-ssh-keys-inside-docker-container/24937401
   
   https://stackoverflow.com/questions/50057180/how-do-i-connect-to-local-machine-from-inside-of-docker-container-created-by-doc



Docker Captain:

* Docker Basics: 

docker container ls  => docker ps
docker container ls  -a 

docker container run => docker run
				   port forwarding  as daemon			  image       CMD
docker container run —publish 80:80 —detach —name webhost ngnix:1.11 ngnix -T
=> webhost comes under names in docker ps

docker container top webhost 

docker container —help


* to remove:

docker container rm id id id

when says being used 
can use -f or docker container stop


* commands to see what is going on inside containers:

docker container top
  - process list in one container 
  -  docker container top webhost
 
docker container inspect
  - details of the container config

docker container stats
  - performance stats for all containers

* Getting inside a container and shell :
 
docker container run -it
   - interactively	(27)						(override default command)
   -  docker container run -it —name proxy nginx bash

Start an existing container:
  - docker container start -ai ubuntu

docker container exec -it
  - run additional commands inside existing running containers
  - docker container exec -it mysql bash
  
 * Network Concepts:   my ip adds => ifconfig en0 (28)
       - docker container port webhost
         - list the ports which are being forwarded
       
      - To get specific information like IP, need to the format from inspect command
         - docker container inspect --format '{{ .NetworkSettings.IPAddress  }}' analytics20-frontend_policy-analytics_1 

  * Docker Networks
    - docker network ls
    - docker network inspect  bridge
    - docker network create — driver  (bridge, host, none)
    - docker network connect network id container id
    - docker network disconnect 

  * DNS and how container find Each Other
     - docker container run -d —name my_nginx —network my_app_net ngnix

  * Network-alias
    - docker container run -d —net dude —net-alias search ubuntu
  
  * Memory

  * https://dzone.com/articles/docker-container-resource-management-cpu-ram-and-i
  * https://linuxhint.com/docker_compose_memory_limits/
  * https://stackoverflow.com/questions/44533319/how-to-assign-more-memory-to-docker-container
