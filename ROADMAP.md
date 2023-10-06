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
