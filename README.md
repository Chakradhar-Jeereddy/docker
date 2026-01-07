# docker
- https://joindevops.medium.com/why-containerization-is-popular-how-to-understand-the-advantages-55261a17517c
- Docker engine is required to run containers

Installation link
==
- https://docs.docker.com/engine/install/rhel/
- Commands
```
sudo dnf -y install dnf-plugins-core
sudo dnf config-manager --add-repo https://download.docker.com/linux/rhel/docker-ce.repo
sudo dnf install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
sudo systemctl start docker
sudo systemctl enable docker
sudo systemctl status docker
```
- After the installation, a docker group is created, only users inside docker group and root user can access docker cli.
- Add ec2-user to docker group to run the commands from that user.
```
sudo usermod ec2-user -aG docker
id ec2-user
uid=1001(ec2-user) gid=1001(ec2-user) groups=1001(ec2-user),990(docker)
```
- Docker home repository is /var/lib/docker

commands
==
```
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
```
Use of --no-cache during the build
==
- Think of Docker build like cooking
  
- Normal docker build (with cache)
- Imagine you cooked rice and curry yesterday.
- Today you cook again:
- Rice is already cooked and kept in the fridge
- Curry is already prepared
- You just reheat them and eat
- Fast
- But rice/curry may not be fresh

- docker build (with no cache)
- You throw away yesterday’s food
- Buy fresh vegetables
- Cook everything from scratch
- Takes more time
- But food is 100% fresh
- Sec team asks to rebuild image with latest system packages and dependencies
- Make sure nothing old remains.
***To get into container***
```
docker exec -it container-id bash
it - interactive mode
```
***To inspect container***
```
docker inspect container-id
- To inspect image
docker inspect image
```
***Note:*** - We can't remove an image being used by container even with force option, when container is running.
            - Stop the container and force remove the image.
            - After force removal of image, container still remains because, it doesn't need image after creation, it stores in its own filesystem layer.
            - Find the container using the image
            - docker ps -a --filter ***ancestor=nginx** or imageid

***Images:*** - AMI = OS + System Packages + App run time + App code + App libraries
              - Docker image = Bare minimum OS + System Packages + App run time + App code + App libraries
              - Image is static, container is running instace of image.


***Dockerfile:*** Instructions to build custom docker images.
                - URL: https://docs.docker.com/reference/dockerfile/
                - Docker file references

FROM
==
Refers to base OS of the image. The dockerfile first instruction should start with FROM.
```
FROM almalinux:9
docker build -t <name> .   (if we don't give version tag it by default takes latest)
docker images
IMAGE          ID             DISK USAGE   CONTENT SIZE   EXTRA
from:latest    0c64b8cdb5a0        272MB         69.7MB
```
PUSH
==
```
docker login -u joindevops
Login succeeded
docker push <URL>/<Username>/<image-name>:<version> (this format is to bring uniquness)
docker push docker.io/chakradhar06/<name>
Using default tag: latest
The push refers to repository [docker.io/chakradhar06/<name>]
tag does not exist: chakradhar06/from:latest
```
Add tag
==
```
docker tag from:latest chakradhar05/from:latest  (to push in central repo we need to tag the image)
docker tag image:version <username>/<image_name>:<version>
docker push docker.io/chakradhar05/from:latest
The push refers to repository [docker.io/chakradhar05/from]
82c710a74610: Pushed
fc156b96a2c0: Pushed
latest: digest: sha256:0c64b8cdb5a0e0c2dd955745b840098cef3f946d8f4bcd4ced580c188a1f264e size: 855

***Note:*** If you are pulling the image, first it will check whether it is available in local repo.
            If not, it will pull from central repo.
docker rm -f `docker ps -a -q`
docker rmi -f images `docker images -q`
docker pull joindevops/from
```
RUN
===
```
Run instruction is used to configure the image. like installing packages, configurations, creating user, etc.
FROM almalinux:9 
RUN dnf install nginx -y

cd RUN/Dockerfile
docker build -t --nocache --progress=plain run:v1 .
docker push 
```

CMD
===
```
reffering the base image
configuring 
CMD ["executable", "params1", "params2"]
CMD ["nginx","-g","daemon off"]

FROM run:v1
CMD ["nginx","-g","daemon off"]

- Instruction in CMD should run the container infinite time. THe command we are giving inside CMD should run in foreground, then we should take it into background.

- CMD -> Instruction used to start the container
- RUN -> these instructions executes at the time of image building
- CMD - these instructions executes at the time of container startup
- TO check if process is running in container use
  docker top container_id
  docker exec container_id curl -I http://localhost
  docker logs container_id

***Note:*** Image can have multiple run instructions
- CMD should be only one. if you give multiple CMD only last one is considered.
- 1 warning found (use docker --debug to expand):
- MultipleInstructionsDisallowed:
- Multiple CMD instructions should not be used in the same stage because only the last one will be used (line 3)

```
LABELS
===
```
Adds the metadata to images, used for filtering
docker images -f "label=trainer=shivkumar"
FROM almalinux:9
LABEL course=k8s \
      grade=2 
```
EXPOSE
===
it will not add any functionality to the image/container, it provides information about ports used by container
EXPOSE <port-number>/<protocal> it will not open the port, just for infomration.
docker inspect container_id

COPY
===
```
FROM nginx
RUN rm -rf /usr/share/nginx/nginx.html
COPY nginx.html /usr/share/nginx/nginx.html
Copies the files from workspace to the container.
```
ENV
===
```
ENV KEY=VALUE
ENV COURSE="DOCKER" \
    TRAINER="CHAKRA" \
    DURATION="10HRS" \
```
- These are accessed from inside the container in the applications

ADD
===
ADD is also like COPY
It can directly fetch from internet not only from workspace
It can untar the file directly into image

ENTRYPOINT
=========
- CMD instructions can be overwritten
- docker run -d env:v1 ping facebook.com
- ETNRYPOINT can't be overwritten, if you try it will go and append and it will lead to failure.
```
To test run - docker run cfdbf35252df ping facebook.com
```
BEST PRACRISE
==========
CMD can be used to supply arg to entrypoint, we can overwrite the default args at runtime.

WORKDIR
==========
Creates a directory and switch to it. No need to use change directory(cd)

ARG
=====
ARG can be used as first instruction to pass arguments
```
Dockerfile
ARG VERSION
FROM almalinux:${VERSION:-9}
docker build  -t arg:v1 --build-arg VERSION=8 .
```
ARG VS ENV
==========
1 . ENV is used to supply the key value pairs for the container/run time
2 . ARGS is used to supply the key value pairs during image build time
3. ARG can't be accessed inside the container
4. ENV can be accessed inside the container
5. We can reuse the arguments as environment variables inside container
```
ARG VERSION
FROM almalinux:${VERSION:-9}
ARG course="chakra" \
    subj="docker"
***ENV course=${course} \***
    subj=${subj}
CMD [ "ping","google.com" ]
```
ONBUILD
============
- Imagine you send someone a ready-made cake base.
- On the cake box, you paste a note:
- When you use this cake base,
➜ add cream
➜ add fruits
➜ then bake”
- You don’t do these steps now.
- They happen later, when someone else uses your cake base.
- That sticky note = ONBUILD
```
FROM chakradhar05/onbuild:latest
RUN dnf install nginx -y
RUN rm -rf /usr/share/nginx/html/index.html
ONBUILD COPY index.html /usr/share/nginx/html/index.html
CMD [ "nginx","-g","daemon off;" ]
```
=> create the image, next time if someone try to build this image, the onbuild instructions will get executed and they have to supply
   the index.html file
   
TEST case
====
```
docker build --no-cache --progress=plain -t onbuild .
Build an image using the above as base image
FROM onbuild:latest
docker build --no-cache --progress=plain -t test .

#6 [2/2] ONBUILD COPY index.html /usr/share/nginx/html/index.html
#6 DONE 0.0s
```

Docker cache
==
- It exists in docker's data directory
    - /var/lib/docker/
| Directory    | Purpose                         |
|-------------|----------------------------------|
| overlay2/   | Image layers (build cache)       |
| containers/ | Container data                  |
| images/     | Image metadata                  |
| volumes/    | Volume data                     |
| buildkit/   | Build cache (modern builds)      |
| network/    | Network configs                 |

- Cache is stored in /var/lib/docker/overlay2/ (old)
- Cache is stored in /var/lib/docker/buildkit/ (new)

Check space usage
==
**docker system df**
```
TYPE            TOTAL     ACTIVE    SIZE      RECLAIMABLE
Images          3         0         771.8MB   224.9MB (29%)
Containers      0         0         0B        0B
Local Volumes   0         0         0B        0B
Build Cache     13        0         475.7MB   40.09MB
```

Docker engine details
==
***docker info**
```
Client: Docker Engine - Community
 Version:    29.1.3
 Context:    default
 Debug Mode: false
 Plugins:
  buildx: Docker Buildx (Docker Inc.)
    Version:  v0.30.1
    Path:     /usr/libexec/docker/cli-plugins/docker-buildx
  compose: Docker Compose (Docker Inc.)
    Version:  v5.0.1
    Path:     /usr/libexec/docker/cli-plugins/docker-compose

Server:
 Containers: 0
  Running: 0
  Paused: 0
  Stopped: 0
 Images: 3
 Server Version: 29.1.3
 Storage Driver: overlayfs
  driver-type: io.containerd.snapshotter.v1
 Logging Driver: json-file
 Cgroup Driver: systemd
 Cgroup Version: 2
 Plugins:
  Volume: local
  Network: bridge host ipvlan macvlan null overlay
  Log: awslogs fluentd gcplogs gelf journald json-file local splunk syslog
 CDI spec directories:
  /etc/cdi
  /var/run/cdi
 Swarm: inactive
 Runtimes: io.containerd.runc.v2 runc
 Default Runtime: runc
 Init Binary: docker-init
 containerd version: dea7da592f5d1d2b7755e3a161be07f43fad8f75
 runc version: v1.3.4-0-gd6d73eb8
 init version: de40ad0
 Security Options:
  seccomp
   Profile: builtin
  cgroupns
 Kernel Version: 5.14.0-362.18.1.el9_3.x86_64
 Operating System: Red Hat Enterprise Linux 9.3 (Plow)
 OSType: linux
 Architecture: x86_64
 CPUs: 2
 Total Memory: 709.5MiB
 Name: ip-172-31-0-53.ec2.internal
 ID: 4ae7cc3b-3d4f-40af-bc03-12d4c52ec1c0
 Docker Root Dir: /var/lib/docker
 Debug Mode: false
 Experimental: false
 Insecure Registries:
  ::1/128
  127.0.0.0/8
 Live Restore Enabled: false
 Firewall Backend: iptables
```














