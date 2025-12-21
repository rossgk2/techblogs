# Containers

A quick overview of containers and container images:

- A *container config file* is a sequence of OS-agnostic commands that can be translated by a piece of software called a *container engine* into the appropriate sequence of OS-particular commands.
  - Container config files for the popular container engine "Docker" are called "Dockerfiles". Docker is so popular that most open source container software still uses the Dockerfile format for container config files.
- A *container image instance* is the result acquired by using a container engine to execute the OS-particular commands specified in a container config file. Multiple instances of a given container image can run independently on the same OS. Since each container image instance is allocated its own resources, *contains* its own files, and hosts its own processes, container image instances are called *containers*.
- If we think of an "environment" as being the result of a sequence of OS-particular commands, it is accurate to say that **the power of containers is that a container image can give a platform-agnostic interface to a highly configured piece of software** (like a database with admin permissions and table schemas already configured, for example). 
- If one were trying to automate the setup of their web application dev environment, they would have one container for each of: the database, the backend application running the API, and the frontend running the UI. They would also need an additional tool to orchestrate the initialization of the containers so that they start in the proper order. *Container composition files* are used just for this.
  - In Docker, container orchestration files are called "Docker compose files".
- Sometimes, downloading already built images is much faster than building said images from the container config files. Because of this, there exist *container image repositories* from which users can pull already-built container images. (So from another perspective, the power of containers is actually that you can quickly download the prebuilt results themselves.) When you use a container engine to such as Docker to pull a container image from an online repository, Docker makes sure to give you a version of the image that is configured for your OS.
- Each container managed by a container composition file is either identified within the composition file by (1) a URL, in the case when it is an instance of a shared image that's been published to a *container registry*, or by (2) a Dockerfile, in the case when it is an instance of a user-defined image.


