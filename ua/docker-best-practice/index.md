# 7 практик щоб працювати з Докером на повну


Development was always a way of evolution. The evolution of modern programming development brings a lot of techniques and requirements - its hard to imagine today's programming without high-level frameworks, containers, cloud computing or special data storages (even if they are not necessary). Working with some of them, I would like to share small notes about the containerization, especially with Docker containers.

<!--more-->

Development was always a way of evolution. The evolution of modern programming development brings a lot of techniques and requirements - its hard to imagine today's programming without high-level frameworks, containers, cloud computing or special data storages (even if they are not necessary). Working with some of them, I would like to share small notes about the containerization, especially with Docker containers. 

## 7 best practices for building containers

Kubernetes Engine is a great place to run your workloads at scale. But before being able to use Kubernetes, you need to containerize your applications. You can run most applications in a Docker container without too much hassle. However, effectively running those containers in production and streamlining the build process is another story. There are a number of things to watch out for that will make your security and operations teams happier. This post provides tips and best practices to help you effectively build containers.

### 1. Package a single application per container

A container works best when a single application runs inside it. This application should have a single parent process. For example, do not run PHP and MySQL in the same container: it’s harder to debug, Linux signals will not be properly handled, you can’t horizontally scale the PHP containers, etc. This allows you to tie together the lifecycle of the application to that of the container.

### 2. Properly handle PID 1, signal handling, and zombie processes

Kubernetes and Docker send Linux signals to your application inside the container to stop it. They send those signals to the process with the process identifier (PID) 1. If you want your application to stop gracefully when needed, you need to properly handle those signals. 

### 3. Optimize for the Docker build cache

Docker can cache layers of your images to accelerate later builds. This is a very useful feature, but it introduces some behaviors that you need to take into account when writing your Dockerfiles. For example, you should add the source code of your application as late as possible in your Dockerfile so that the base image and your application’s dependencies get cached and aren’t rebuilt on every build.

Take this Dockerfile as example:
```docker
FROM python:3.5
COPY my_code src
RUN pip install my_requirements
```

You should swap the last two lines:
```docker
FROM python:3.5
RUN pip install my_requirements
COPY my_code src
```
In the new version, the result of the pip command will be cached and will not be rerun each time the source code changes.

### 4. Remove unnecessary tools

Reducing the attack surface of your host system is always a good idea, and it’s much easier to do with containers than with traditional systems. Remove everything that the application doesn’t need from your container. Or better yet, include just your application in a distroless or scratch image. You should also, if possible, make the filesystem of the container read-only. This should get you some excellent feedback from your security team during your performance review.

### 5. Build the smallest image possible

Who likes to download hundreds of megabytes of useless data? Aim to have the smallest images possible. This decreases download times, cold start times, and disk usage. You can use several strategies to achieve that: start with a minimal base image, leverage common layers between images and make use of Docker’s multi-stage build feature.

### 6. Properly tag your images

Tags are how the users choose which version of your image they want to use. There are two main ways to tag your images: Semantic Versioning, or using the Git commit hash of your application. Whichever your choose, document it and clearly set the expectations that the users of the image should have. Be careful: while users expect some tags —like the “latest” tag— to move from one image to another, they expect other tags to be immutable, even if they are not technically so. For example, once you have tagged a specific version of your image, with something like “1.2.3”, you should never move this tag.

### 7. Carefully consider whether to use a public image

Using public images can be a great way to start working with a particular piece of software. However, using them in production can come with a set of challenges, especially in a high-constraint environment. You might need to control what’s inside them, or you might not want to depend on an external repository, for example. On the other hand, building your own images for every piece of software you use is not trivial, particularly because you need to keep up with the security updates of the upstream software. Carefully weigh the pros and cons of each for your particular use-case, and make a conscious decision.
