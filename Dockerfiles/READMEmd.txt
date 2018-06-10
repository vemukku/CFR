
Project Title - “Multip App Container Image”
Install and Configure Nginx as a Web Server and Reverse Proxy for Apache for Php applications using Dockerfile with Centos7 Base Image.
The setup is to have nginx out front serving static assets and then proxy to Apache/mod_php for PHP scripts. 
Apache and Nginx are two popular open source web servers often used with PHP. It can be useful to run both of them on the same virtual machine when hosting multiple websites which have varied requirements. The general solution for running two web servers on a single system is to either use multiple IP addresses or different port numbers.
Nginx listens on port 80 for all web traffic. If it gets a request for something that isn't static (like a .php file, or a RESTful URL pointing to a Ruby/Python/etc. app) it passes it along to Apache, which is listening on another port 8080(perhaps 8080 on the loopback interface).
Anything static (JS, CSS, HTML, image files, etc) nginx serves up on its own, never bothering Apache to spin up a process and do the job.
The only actual reason to use Apache to serve PHP (instead of Nginx and PHP-FPM) is if you're offering a shared-tenancy hosting service, where multiple independent customers are being served out of the same webserver host.  
The setup is to have nginx out front serving static assets and then proxy to Apache/mod_php for PHP scripts. 
If you want to use nginx and PHP then you have no choice but to use FastCGI as nginx doesnt have the equivalent of mod_php.
Why not just use nginx+FastCGI instead of nginx+apache+mod_php?
The LAMP stack (Linux, Apache, MySQL, and PHP) has been a complete web solution to developers for more than a decade.
Apache Drawbacks:
Apache falls short when it comes to serving plenty of static files. However, the latest version of Apache has resolved this issue considerably.  Moreover, Apache, having a process-based mechanism, is definitely a heavy memory web server.
Nginx Drawbacks:
Nginx can only be used with PHP-FPM, and that lacks support for .htaccess. This means a simple redirection can even be a technical issue. Moreover, while Nginx is undoubtedly fast in delivering static content, Apache still leads when it comes to PHP requests.
--------------------------------------------------------------------------------------------------------
Getting Started:
 Prerequisites: 
  Install Docker and Learn Docker Container Manipulation – Part 1
  https://www.tecmint.com/install-docker-and-learn-containers-in-centos-rhel-7-6/
  Deploy and Run Applications under Docker Containers – Part 2
  https://www.tecmint.com/install-run-and-delete-applications-inside-docker-containers/       

 Building a Docker Container(Image) :

Creating a Dockerfile to Automatically Build Nginx+Apache+mod_php Image.

Docker images can be automatically build form text files, named Dockerfile. A Docker file contains step-by-step ordered instructions or commands used to create and configure a Docker image.
A Docker file contains various instructions in order to build and configure a specific container based on your requirements. 
The following instructions are the most used, some of them being mandatory:

FROM = Mandatory as first instruction in a Docker file. Instructs Docker to pull the base image from which you are building the new image. Use a tag to specify the exact image from which you are building:
Ex:- FROM centos:7 ( https://hub.docker.com/_/centos/)
MAINTAINER = Author of the build image
RUN = This instruction can be used on multiple lines and runs any commands after Docker image has been created.
CMD = Run any command when Docker image is started. Use only one CMD instruction in a Dockerfile.
ENTRYPOINT = Same as CMD but used as the main command for the image.
EXPOSE = Instructs the container to listen on network ports when running. The container ports are not reachable from the host by default.
ENV = Set container environment variables.
ADD = Copy resources (files, directories or files from URLs).
COPY=The COPY instruction copies new files or directories from <src> and adds them to the filesystem of the container at the path <dest>.

Steps:

First, let’s create some kind of Dockerfile repositories in order to reuse files in future to create other    
images. Make an empty directory somewhere in /var partition where we will create the file with the 
instructions that will be used to build the newly Docker image.

mkdir -p /var/docker/centos/Dockerfiles
touch /var/docker/centos/Dockerfiles/Dockerfile

Next, start editing the file with the following instructions: https://github.com/vemukku/Multi_Services/blob/CFR/Dockerfiles/Dockerfile

Once created then validate the Dockerfile 
https://www.fromlatest.io/#/
http://hadolint.lukasmartinelli.ch/
https://www.npmjs.com/package/validate-dockerfile

Build Image :

Syntax: 
docker build [OPTIONS] PATH | URL | -

From text file:

$ docker build –t  centos7-multiapp: nginxapache  –f  /var/docker/centos/Dockerfiles/Dockerfile

Image Name = centos7-multiapp  
Tag= nginxapache  

If you are in same directory where Dockerfile exist then:

$ docker build -t centos7-multiapp:nginxapache  Dockerfile

-t = tag, it specifies the tag to identify the image, the form is "imagename:tag".
-f = file, used to specify the dockerfile to use, e.g. -f Location of Dockerfile

Note:
Run in current directory, only required parameter. This affects the default location of the Dockerfile and the current directory in the context of building the image from the Dockerfile (e.g., the location from which to resolve relative paths in the Dockerfile).
NOTE: Re-running the same build command after changing the dockerfile will create a new image and give the new image the tag specified, which will result in the old image losing the specified repository and tag. The old image is not removed and will still be accessible via its Image ID.

From Git repositories:

$ docker build -t centos7-multiapp      
https://github.com/vemukku/CFR-Multi-Services-Dockerfile/blob/master/Dockerfile
https://github.com/vemukku/CFR-Multi-Services-Dockerfile:Dockerfile

From Tar:

$ docker build http://server/context.tar.gz
$ docker build -f ctx/Dockerfile http://server/ctx.tar.gz

After the image has been created by Docker, you can list all available images and identify your   
 image by issuing the following command:

 $docker images
 $docker ps

 Run the Container :
 Create a new container based on centos7-multiapp . And before create new container, we can    
 create new directory on the host machine.

 mkdir -p /images
Now run the new container with command below:

$ docker run –d -t -i --name centos7-multiapp: nginxapache  -p 80:80 –p 8080:8080 --mount type=bind,src= /nginx Document Root path on host/,dst= /nginxpath in container/ --mount type=bind,src= /apache document root path/on/host/,dst= /apache path/in/container/


 -p 80:80 =nginx running  on port 80 
 -p 8080:8080 = apache running apache  port 8080 

Inspect Image which we created:
DockerFiles>$ docker inspect  imagename – pretty

Push the Image to Remote Github or Registry:

$docker login
$docker push repository URL//imagename:tag

Add bind-mounts or volumes:
Docker supports two different kinds of mounts, which allow containers to read to or write from files or directories on other containers or the host operating system. These types are data volumes (often referred to simply as volumes) and bind-mounts.
A bind-mount makes a file or directory on the host available to the container it is mounted within. A bind-mount may be either read-only or read-write. For example, a container might share its host's DNS information by means of a bind-mount of the host's /etc/resolv.conf or a container might write logs to its host's /var/log/myContainerLogs directory. If you use bind-mounts and your host and containers have different notions of permissions, access controls, or other such details, you will run into portability issues.
A named volume is a mechanism for decoupling persistent data needed by your container from the image used to create the container and from the host machine. Named volumes are created and managed by Docker, and a named volume persists even when no container is currently using it. Data in named volumes can be shared between a container and the host machine, as well as between multiple containers. Docker uses a volume driver to create, manage, and mount volumes. You can back up or restore volumes using Docker commands.
Consider a situation where your image starts a lightweight web server. You could use that image as a base image, copy in your website's HTML files, and package that into another image. Each time your website changed, you'd need to update the new image and redeploy all of the containers serving your website. A better solution is to store the website in a named volume which is attached to each of your web server containers when they start. To update the website, you just update the named volume.

 
--------------------------------------------------------------------------------------------------
    

