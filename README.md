# docker
https://joindevops.medium.com/why-containerization-is-popular-how-to-understand-the-advantages-55261a17517c
-For running container we need docker engine
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

commands 
docker ps -> All running containers
docker ps -a -> All containers

Images
AMI = OS + System Packages + App run time + App code + App libraries
docker image = Bare minimum OS + System Packages + App run time + App code + App libraries
image is static, container is running instace of image.

docker pull nginx -> pulling the image from docker hub
docker pull nginx:perl -> pull specific version, default is latest
docker images -> shows images stored locally
docker run ngix(runs in foreground) = docker pull ngix-> docker create nginx:latest -> docker start nginx
docker run -d nginx (detached mode,runs in background)

docker stop container-id
docker rm container-id
docker rm container-id -f (force remove running container)
docker rmi container-id










