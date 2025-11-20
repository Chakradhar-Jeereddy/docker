# docker
https://joindevops.medium.com/why-containerization-is-popular-how-to-understand-the-advantages-55261a17517c
-For running container we need docker engine

Installation
https://docs.docker.com/engine/install/rhel/
```
sudo dnf -y install dnf-plugins-core
sudo dnf config-manager --add-repo https://download.docker.com/linux/rhel/docker-ce.repo
sudo dnf install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
sudo systemctl start docker
sudo systemctl enable docker
sudo systemctl status docker
```
A docker group is created, only users inside docker group and root user can access docker cli.
Add ec2 user to docker group
```
sudo usermod ec2-user -aG docker
id ec2-user
uid=1001(ec2-user) gid=1001(ec2-user) groups=1001(ec2-user),990(docker)
```
Docker home repository is /var/lib/docker

commands 
docker ps -> All running containers
docker ps -a -> All containers
docker pull nginx -> pulling the image from docker hub
docker pull nginx:perl -> pull specific version, default is latest
docker images -> shows images stored locally
docker run ngix(runs in foreground) = docker pull ngix-> docker create nginx:latest -> docker start nginx
docker run -d nginx (detached mode,runs in background)
docker inspect <contanerid>
docker inspect <imageid>
docker exec -it containerid /bin/bash
docker exec -it 8b9adf6bad08 /bin/bash
docker stop container-id
docker rm container-id
docker rm container-id -f (force remove running container)
docker rmi container-id

Port forwarding (first reaches host and then container), host port is unique
docker run -d  -p <host-port>:<container-port> nginx
docker run -d -p 80:80 nginx
docker run -d -p 8080:80 nginx

To get into container
docker exec -it container-id bash
it - interactive mode

to inspect container
docker inspect container-id
to inspect image
docker inspect image

Images
AMI = OS + System Packages + App run time + App code + App libraries
docker image = Bare minimum OS + System Packages + App run time + App code + App libraries
image is static, container is running instace of image.


Dockerfile: Instructions to build custom docker images.
URL: https://docs.docker.com/reference/dockerfile/
Docker file references

FROM
######################
Refers to base OS of the image. The dockerfile first instruction should be FROM
FROM almalinux:9

docker build -t from .   (if we don't give version tag it by default takes latest)
docker images
IMAGE          ID             DISK USAGE   CONTENT SIZE   EXTRA
from:latest    0c64b8cdb5a0        272MB         69.7MB

PUSH
#######################
docker login -u joindevops
Login succeeded
docker push <URL>/<Username>/<image-name>:<version> (this format is to bring uniquness)
docker push docker.io/chakradhar06/from
Using default tag: latest
The push refers to repository [docker.io/chakradhar06/from]
tag does not exist: chakradhar06/from:latest

Add tag
docker tag from:latest chakradhar05/from:latest
docker push docker.io/chakradhar05/from:latest
The push refers to repository [docker.io/chakradhar05/from]
82c710a74610: Pushed
fc156b96a2c0: Pushed
latest: digest: sha256:0c64b8cdb5a0e0c2dd955745b840098cef3f946d8f4bcd4ced580c188a1f264e size: 855

If you are pulling the image, first it will check whether it is available in local repo.
If not, it will pull from central repo.

docker rm -f `docker ps -a -q`
docker rmi -f images `docker images -q`

docker pull joindevops/from

RUN
===
Run instruction is used to configure the image. like installing packages, configurations, creating user, etc.
FROM almalinux:9 
RUN dnf install nginx -y

cd RUN/Dockerfile
docker build -t --nocache --progress=plain run:v1 .
docker push 


CMD
===
reffering the base image
configuring 
CMD ["executable", "params1", "params2"]
CMD ["nginx","-g","daemon off"]

FROM run:v1
CMD ["nginx","-g","daemon off"]

Instruction in CMD should run the container infinite time. THe command we are giving inside CMD should run in foreground, then we should take it into background.
CMD -> Instruction used to start the container
RUN -> these instructions executes at the time of image building
CMD - these instructions executes at the time of container startup

Image can have multiple run instructions
CMD should be only one. if you give multiple CMD only last one is considered.

LABELS
===
Adds the metadata to images, used for filtering
docker images -f "label=trainer=shivkumar"

EXPOSE
===
it will not change any functionality to the image/container.

EXPOSE 80/tcp

COPY
===
FROM nginx
RUN rm -rf /usr/share/nginx/nginx.html
COPY nginx.html /usr/share/nginx/nginx.html
COPY qi/ /usr/share/nginx/

ENV
===
ENV KEY=VALUE
ENV COURSE="DOCKER" \
    TRAINER="CHAKRA" \
    DURATION="10HRS" \

ADD
===
ADD us also like COPY
It can directly fech from internet not only from workspace












