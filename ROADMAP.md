# Docker: Up and Running

As mentioned in `README.md`, we will be following along with the book "Docker: Up and Running" by Sean P. Kane. This roadmap is meant to explain the main concepts discussed in the book, and how they are implemented here.<br>
Also, since the idea it to take a hands-on approach, we will start at chapter 4.

## Chapter 04: Working with Docker Images
### 4.1: Docker Images
Images are the blueprint from which containers are built. In trun, images are built from Dockerfiles.
Dockerfiles list the steps required to build an image. One can run as many containers as one would like from the same image.

Dockerfile --> Image --> Container

Each line in the Dockerfile corresponds to a *layer*. This is great because it allows us to chaché some of the building process. When a Dockerfile is changed, the image is rebuilt. But, Docker only needs to rebuild the layers that have changed. 
So it is usually a good idea to put all the heavier layers at the start of the Dockerfile, and the lighter (or more likely to change) layers at the end.

#### 4.1.1: Dockerfiles
With the dockerfile at `docker-node-hello/Dockerfile` as an example, we can get a feel for what each part does.

---
```dockerfile
FROM docker.io/node:18.13.0
```
This pulls a base image from Docker Hub. This is like a blank canvas. We will then modify this image to suit our needs.
---
```dockerfile
ARG email="anna@example.com"
```
Defining ARGs allows us to pass arguments into the image. We can later use the ARG like so: `$email`
---
```dockerfile
ENV AP /data/app
ENV SCPATH /etc/supervisor/conf.d
```
The ENV command allows us to define environment variables to be then used when running shell commands.
---
```dockerfile
RUN apt-get -y install supervisor
RUN mkdir -p /var/log/supervisor
```
The RUN command allows us to install packages, create directories, etc. We can use it to run commands as if we were in a shell
---
```dockerfile
COPY ./supervisord/conf.d/* $SCPATH/
```
With the COPY command, one can copy files from the host machine to the image. Once the image is built, there is no need to access the host machine's filesystem, since the files are already in the image.
---
```dockerfile
CMD ["supervisord", "-n"]
```
The CMD command is the command that is executed when the container starts. Once the command is executed, the container exits.
This is why we usually see commands that start long-running processes.


#### 4.1.2: Build an image
From the `Dockerfile`, one can build the image with the `docker image build` command.
```
docker image build -t example/docker-node-hello:latest .
```

In this case, the `-t` flag is used to name (or *tag*) the image `example/docker-node-hello:latest`

#### 4.1.3: Running an image
We can run containers with the image we just built with the command: `docker container run --rm -d -p 8080:8080 <image_name>`. In this case, the name is the one we defined with the `-t` flag in the build stage, `example/docker-node-hello:latest`.
In this *run* stage, we used a few flags:
- `--rm` is used to remove (delete) the container after it is stopped
- `-d` means it is run in *detached* mode. This means that the container will run in the background.
- `-p` is used to map ports inside the container to ports in the host machine. In this case, the port `8080` on the host to the port `8080` in the container

#### 4.1.4: Inspecting an image
One can inspect and get information about a Docker image with the `docker image inspect` command.
In our case:
`docker image inspect example/docker-node-hello:latest`.
Also, the `history` command can be used to get information about the layers that make up the image. In our case: `docker image history example/docker-node-hello:latest`. 
This will show the layers that make up the image, as well as the commands that were run to create each layers, and the size of each layer. This command can come in handy when optimizing Docker images

#### 4.1.5: Environment variables
We can pass environment variables to the container with the `--env` (or `-e`) flag. For example: `sudo docker run --rm -d --env WHO='Panzi' -p 8080:8080 example/docker-node-hello:latest`

By passing the `-env WHO='Panzi'` flag, a new `WHO` environment variable is created in the container, with the value `Panzi`. Then the app will use the environment variable to greet `Panzi` instead of `World`. (For more details, one can see the `docker-node-hello/index.js` file)


#### 4.1.6: Misc
- *Alpine* images are smaller than *Ubuntu* images. This is because Alpine images are built with a smaller base image. This is why they are usually preferred for production environments. However, there a few trade-offs that one has to be aware of. For example, Alpine images do not have `bash` installed by default. So, if one needs to run a container with `bash`, one would have to install it first. Also, there are a few details related to JVM-related apps and DNS resolution.
- **Order matters** when it commes to the commands one runs in the Dockerfile. It is usually a good idea to put the commands that are more likely to change at the end of the Dockerfile, so that Docker can leverage the cache and not have to rebuild the whole image every time a change is made.
In the words of the book:
> *The important lesson to take away from all of this is that order matters, and in general, you should always try to order your Dockerfile so that the most stable and time-consuming portions of your build process happen first and your code is added as late in the process as possible.*
- In term os cache, one can also leverage directory caching. This is generally used for package manager's caches.

### 4.2: Image Stores
Images are not generally built on production servers. Instead, they are built somewhere else, then stored, and then downloaded by the servers that will run one or more instances of the image
There are a few options available for storing images:
- Public registries: Docker Hub, Quay.io, etc. They are *like GitHub* for docker images
- Private registries: Docker offers a "built-in" solution. However, there are a few open source tools that offer a UI to navigate private registries. 


In personal projects, one will mos likely interact with DockerHub. One **important** detail is that the image must have contain the same name a the login user. Let me explain:
Let's take, for example, the image we built: `example/docker-node-hello`. When trying to push the iage to the public registry, we will get an error, because this will resolve to something like `docker.io/example/docker-node-hello`. In order for it to work as we expect it to work, it should resolve to something like `docker.io/<user>/docker-node-hello`. This is similar to GitHub, where the repository links are something like `github.com/<user>/<repo_name>`
We can create an image clone with the correct name with the `docker image tag` command. 

Once the image is correctly named, we can upload it to the registry with the `docker image push <image-name>` command.
To use the image on another host, one can pull it with the `docker image pull <image-name>` command.
This is very similar to GitHub repositories!

### 4.3: Troubleshooting:
There are many ways to troubleshoot Docker images whe the build goes wrong. Usually, Google (or StackOverflow, or ChatGPT) will guide you to the right solution. However, there are a few things that one can do to troubleshoot the build process. 
One thing that really helped me understand the build process is to notice that image layers are images themselves. These layers can be ran as containers. This is very useful when trying to debug the build process. For example, if one wants to see what is inside a layer, one can run it as a container with the `docker container run -it <layer-name> bash` command. This will run the layer as a container, and open a bash shell inside the container. From there, one can run commands to inspect the layer, and see what is going on.
It is worth noting that if `BuildKit` is enabled, we won't be able to approach the debugging process in the same way. In this case, multistage builds can be leveraged to reproduce the error and understand what's happening.


## Chapter 05: Working with Containers
In the last chapter we saw a way to run containers from an image, with the `docker container run` command.
The `run` command actually executes two sequential commands `create` and `start`.
The `create` command creates a container from an image, and the `start` command starts the container.

### 5.1: Useful Flags
- `-d` or `--detach`: runs the container in the background
- `-p` or `--publish`: maps ports from the container to the host
- `-e` or `--env`: sets environment variables
- `--name=<name>`: sets the name of the container
example: `docker container create --name="awesome-service" ubuntu:latest sleep 120`
- with the `sleep 120` arguments, the container will run for 120 seconds, and then exit. This is useful for testing purposes. `sleep 120` is actually the command ran by the container!
- The `-l` flag is used to attach labels to the container. This is useful for filtering containers with the `docker container ls --filter label=<label>` command. Example: `docker container ls -a -f label=app=awesome`
Labels are metadata that can be used to find containers easily
example: `docker container create --name awesome-service -l app=awesome -l env=dev ubuntu:latest`
- `-i` is used to run the container interactively. It is usually combined with the `-t` flag and the `/bin/bash` command. `-i` tells the container to keep STDIN open, and `-t` tells the container to allocate a pseudo-TTY (a terminal-like interface).
example: `docker run --name=ubutest -l test=true -it ubuntu:latest /bin/bash`

#### 5.1.1: Volumes and Storage
Volumes and storage can be managed through the `-v` and `--mount` flags. Most of the time one will use the shorthand `-v` flag. However, `--mount` provides with finer-grained control over the volume.
example: 
```
docker container run -it --rm --name=ubutest -v /mnt/host_data:/container_data ubuntu:latest /bin/bash
```
Running the above command will result in a binding between the directory `/mnt/host_data` in the host machine, and the directory `/container_data` in the container. This means that any changes made to the `/container_data` directory in the container will be reflected in the `/mnt/host_data` directory in the host machine, and vice-versa.
This might be a problem if we don't want the process running inside the container to write to the host machine. In this case, we can use the `--read-only` flag to make the volume read-only.
Also, using the `--mount` flag we will be able to further configure the volumes mounted.
example:
```
docker container run --rm -ti --read-only=true \
--mount type=bind,target=/mnt/session_data,source=/data \
ubuntu:latest /bin/bash
```

#### 5.1.2: Resource Management: CPU
- The `--cpu-shares` flag is used to set the CPU shares for the container. The maximum value (which is also the default value) is 1024. 1024 would mean that the container is not restricted, it can use as much CPU as it needs. A value of 512 would mean that the container is capped, and it is restricted to using only half of the computing power
- The `--cpuset-cpus=0-2` flag is use for hard sharding. This example pins the container to the first 2 CPU cores (cores 0 and 1).*"In this command, the `--cpuset-cpus=0,1` option specifies that these CPUs are CPU 0 and CPU 1 on the host system"*
- The `--cpus` flag is a way to simplify the control over CPU resource allocation. One can pass a float between 0.01 and the max number of CPUs in the host machine to control the resources each container can use.

#### 5.1.3: Resource Management: Memory
The memory is allocated trhough the `--memory` flag. There is a way to control how much memory and swap-memory is allocated to the container, but we won't cover it here. 
If we allocate less memory than needed by the container, the container (actually, the process running in the container) will fail.

**One important distinction the book makes is that memory management is a hard cap, while CPU management is just a priority setting.**

#### 5.1.4: Misc
- passing the `--restart=on-failure:3` flag will restart the container if it fails. It will try to restart it 3 times before giving up.

### 5.2: Stopping containers
- `docker container stop` stops the container and sends a `SIGTERM` signal to the process running inside the container. i.e. it terminates the process.
- `docker container kill` is usually used when something is not working properly. It sends a `SIGKILL` signal to the process running inside the container. i.e. it kills the process. 
- `docker container pause` just pauses the container. It does not exit, since the process is not terminated.

## Chapters 06 and 07: Inspection and Debugging
I will not cover these chapters in detail here, though I recommend reading them. They cover a few useful commands that can be used to inspect and debug containers. 
However, I feel like inspecting and debugging containers is something that is better learned by doing. The chapters provide general guidelines, but most debugging will be application-specific.
As one gets more experience with Docker, one will naturally get more familiar with the tools and commands that are useful for debugging.
