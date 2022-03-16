# Docker Best Practices

Docker is a high-level virtualization platform that lets you package software as isolated units called containers.
These tips are based on books, articles and professional experience.

## Table of Contents

1. [Update Docker version regularly](#update-docker-version-regularly)
2. [Use official images](#use-official-images)
3. [Prefer distroless images](#prefer-distroless-images)
4. [Prefer fast start-up times](#prefer-fast-start-up-times)
5. [Use specific image versions](#use-specific-image-versions)
6. [Use small-sized images](#use-small-sized-images)
7. [Update images frequently](#update-images-frequently)
8. [Scan images regularly](#scan-images-regularly)
9. [Sign and verify images](#sign-and-verify-images)
10. [Decouple applications](#decouple-applications)
11. [Use .dockerignore file](#use-dockerignore-file)
12. [Exclude unnecessary packages](#exclude-unnecessary-packages)
13. [Sort multi-line arguments](#sort-multi-line-arguments)
14. [Review the Dockerfiles](#review-the-dockerfiles)
15. [Pipe Dockerfile through stdin](#pipe-dockerfile-through-stdin)
16. [Understand CMD vs ENTRYPOINT](#understand-cmd-vs-entrypoint)
17. [Use COPY instead of ADD](#use-copy-instead-of-add)
18. [Use metadata labels](#use-metadata-labels)
19. [Use multi-stage builds](#use-multi-stage-builds)
20. [Use layer caching](#use-layer-caching)
21. [Minimize the number of layers](#minimize-the-number-of-layers)
22. [Build one image for all environments](#build-one-image-for-all-environments)
23. [Use the least privileged user](#use-the-least-privileged-user)
24. [Make executables owned by root and not writable](#make-executables-owned-by-root-and-not-writable)
25. [Prevent confidential data leaks](#prevent-confidential-data-leaks)
26. [Include health/liveness checks](#include-healthliveness-checks)
27. [Segregate container networks](#segregate-container-networks)
28. [Limit container resources](#limit-container-resources)
29. [Avoid privileged containers](#avoid-privileged-containers)
30. [Set group ownership and file permissions](#set-group-ownership-and-file-permissions)
31. [Run Docker in rootless mode](#run-docker-in-rootless-mode)
32. [Protect the Docker daemon socket](#protect-the-docker-daemon-socket)
33. [Protect the Docker host](#protect-the-docker-host)
34. [Maintain host isolation](#maintain-host-isolation)
35. [Filter system calls](#filter-system-calls)
36. [Do not map any ports below 1024](#do-not-map-any-ports-below-1024)
37. [Set PID limits](#set-pid-limits)
38. [Shutdown smartly and gracefully](#shutdown-smartly-and-gracefully)
39. [Ensure that containers are stateless and immutable](#ensure-that-containers-are-stateless-and-immutable)
40. [Secure containers at runtime](#secure-containers-at-runtime)
41. [Monitor system resources](#monitor-system-resources)
42. [Monitor container activity](#monitor-container-activity)
43. [Configure logging mechanisms](#configure-logging-mechanisms)
44. [Use a secrets management tool](#use-a-secrets-management-tool)
45. [Use a private registry](#use-a-private-registry)
46. [Use Docker Compose](#use-docker-compose)
47. [Use a linter](#use-a-linter)
48. [Use a build tool](#use-a-build-tool)
49. [Use Docker Lock](#use-docker-lock)
50. [Have an alert system](#have-an-alert-system)

## Update Docker version regularly

Ensure that your Docker version is up to date. Obsolete versions are susceptible to security attacks.
New version releases often contain patches and bug fixes that address vulnerabilities of older versions.
Note that to install the latest version, you must first purge the current installation.
The same holds true for the host environment.
Ensure that supporting applications are up-to-date and free of known bugs or security loopholes.

## Use official images

When you're extending or building on top of another image, choosing a well-known and well-maintained base image is essential.
Let's say you are developing a Node.js application and want to build and run it as a Docker image.
Instead of taking a base operating system image and installing Node.js and whatever other tools you need for your application, use the [official Node.js image](https://hub.docker.com/_/node) for your application.
The [Docker Official Images](https://hub.docker.com/search?q=&type=image&image_filter=official) are a curated set of Docker repositories hosted on Docker Hub.
Official images are created with the best practices that affect the versatility, security, and efficiency of your container.

## Prefer distroless images

You can use the [distroless](https://github.com/GoogleContainerTools/distroless) images from Google's distroless project.
These images are based on Debian and have absolute minimum tools installed, reducing the overall potential surface for attack.
They do not contain package managers, shells, or any other programs in a standard Linux distribution.
Restricting what's in your runtime container to precisely what's necessary for your app is a best practice employed by Google and other tech giants that have used containers in production for many years.
If you don't want to use one of the community base images, consider using a minimal Linux base image like [Alpine](https://hub.docker.com/_/alpine).

## Prefer fast start-up times

Making sure your application started quickly has always been a goal.
But it's even more important in containers, which are often run at scale, and moved around between different servers.
To achieve this, for example, prefer small-sized base images, exclude unnecessary packages or libraries, prefer small-sized dependencies, copy only necessary files to the container, use multi-stage builds, use Ahead of Time compilation when possible, be aware of any tweaks you might make to your framework, etc.

## Use specific image versions

Tags are commonly used to manage versions of Docker images.
For example, a `latest` tag is used to indicate that this is the latest version of an image.
However, image tags are not immutable, and the author of the images can publish the same tag multiple times, causing confusion and inconsistent behaviour in automated builds.
Whenever possible, avoid using the `latest` tag.
You can pin image tags using a specific image version using `major.minor`, not `major.minor.patch`, to ensure you are always keeping your builds working and getting the latest security updates.
Alternatively, SHA pinning gives you completely reliable and reproducible builds, but it also likely means you won't have any obvious way to pull in important security fixes from the base images you use.

## Use small-sized images

You see in Docker Hub multiple images, not only with different version numbers, but also with different operating systems.
If the image is based on a full-blown OS distribution like [Ubuntu](https://hub.docker.com/_/ubuntu) or [Centos](https://hub.docker.com/_/centos), you will have a bunch of tools already packaged in the image.
The image size will be larger, but you don't need most of these tools in your application images.
In comparison by using smaller images with leaner OS distributions, which only bundle the necessary system, you need less storage space, minimize the attack surface, and ensure you create more secure images.
The best practice here would be to select an image with a specific version based on a leaner OS distribution like [Alpine](https://hub.docker.com/_/alpine).

## Update images frequently

Use base images that are frequently updated and rebuild yours on top of them.
As new security vulnerabilities are discovered continuously, it is a general security best practice to stick to the latest security patches.
There is no need to always go to the latest version, which might contain breaking changes, but define a versioning strategy.
You can stick to stable or long-term support versions, which deliver security fixes soon and often.
Also, rebuild your images periodically with a similar strategy to get the latest packages from the base distro, Node, Golang, Python, etc.

## Scan images regularly

When you choose a base image for your Docker container, you indirectly assume the risk of all the container security concerns that the base image is bundled with.
[Image scanning](https://docs.docker.com/develop/scan-images/) is way of detecting potential problems before running your containers.
It is a security best practice to apply the "shift left security" paradigm by directly scanning your images, as soon as they are built, in your CI pipelines before pushing to the registry.
[Docker and Snyk](https://snyk.io/docker/) have partnered together to bring security natively into the development workflow by providing a simple and streamlined approach for developers to build and deploy secure containers.
[Clair](https://github.com/quay/clair), [Trivy](https://github.com/aquasecurity/trivy) and [Docker Bench for Security](https://github.com/docker/docker-bench-security) are alternatives to Snyk.

## Sign and verify images

Docker allows signing images, and by this, provides another layer of protection.
To sign images, use [Docker Content Trust](https://docs.docker.com/engine/security/trust/).
Docker Content Trust provides the ability to use digital signatures for data sent to and received from remote Docker registries.
These signatures allow client-side or runtime verification of the integrity and publisher of specific image tags.
Within the Docker CLI you can sign and push a container image with the docker trust command syntax.
This is built on top of the [Notary](https://github.com/notaryproject/notary) feature set.
To verify the integrity and authenticity of an image, set `DOCKER_CONTENT_TRUST=1` environment variable.
Therefore, if you try to pull an image that hasn't been signed, you'll receive an error.

## Decouple applications

Containerisation is about [creating a single unit of deployment, or a single concern](https://www.tutorialworks.com/containers-single-or-multiple-processes/).
Each container should have just one program running inside it.
Decoupling applications into multiple containers makes it easier to scale horizontally and reuse containers.
Use your best judgment to keep containers as clean and modular as possible.
If containers depend on each other, you can use Docker container networks to ensure that these containers can communicate.

## Use .dockerignore file

When you build an image, you don't need everything you have in the project to run the application inside.
You don't need the auto-generated folders, like `targets`, `build` or `dist`, the `README` file, etc.
The `.dockerignore` file tells Docker to not copy unwanted files into the image.
This file is used to specify the files and folders that you don't want to be added to the build context.
Put another way, make use of it to keep the files in the final image to a minimum.
It is certainly not essential, but it will make your images quicker to pull and run.
Also, small images need less storage space and will minimize the attack surface.

## Exclude unnecessary packages

Excluding unnecessary packages and build dependencies from the final image will reduce the image size and will minimize the attack surface.
When you build an app with tools like [Maven](https://maven.apache.org/) or [npm](https://www.npmjs.com/), they tend to download a bunch of dependencies.
If you release this image, it will contain a lot of unnecessary dependencies.
To exclude all those packages from your final image, you can copy only the files and folders that you want to add to the release image, and/or install only production dependencies (e.g. via `npm ci --only=production` for Node.js).
Also, you can use a Docker multi-stage build.

## Sort multi-line arguments

Whenever possible, ease later changes by sorting multi-line arguments alphanumerically.
This helps to avoid duplication of instructions and make the list much easier to update.
This also makes Pull Requests a lot easier to read and review. See the next example:

```dockerfile
RUN apt-get update && apt-get install -y \
  git \
  mercurial \
  subversion \
  && rm -rf /var/lib/apt/lists/*
```

## Review the Dockerfiles

Code Review, also known as Peer Review, is the act of consciously and systematically convening with one's fellow programmers to check each other's code for mistakes and has been repeatedly shown to accelerate and streamline the process of software development.
Dockerfiles are code too. The content of a Dockerfile can be critical, so you should always peer-review the Dockerfiles, just as you do for your application code.
Also, if you have your Dockerfiles in a separate repository, it is recommended to follow CI/CD practices to build the respective Docker images.

## Pipe Dockerfile through stdin

Docker can build images by piping Dockerfile through `stdin` with a local or remote build context.
[Piping a Dockerfile through stdin](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/#pipe-dockerfile-through-stdin) can be useful to perform one-off builds without writing a Dockerfile to disk, or in situations where the Dockerfile is generated, and should not persist afterwards.
See the next example:

```bash
docker build -<<EOF
FROM busybox
RUN echo "hello world"
EOF
```

## Understand CMD vs ENTRYPOINT

CMD and ENTRYPOINT are two instructions used to define the process that's run in your container.
The [CMD](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/#cmd) instruction should be used to run the software contained in your image, along with any arguments.
CMD should almost always be used in the form of `CMD ["executable", "param1", "param2"…]`.
[ENTRYPOINT](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/#entrypoint) sets the process that's executed when the container first starts.
The best use for ENTRYPOINT is to set the image's main command, allowing that image to be run as though it was that command (and then use CMD as the default flags).
CMD commands are ignored by Daemon when there are parameters stated within the `docker run` command.
ENTRYPOINT instructions are not ignored but instead are appended as command line parameters by treating those as arguments of the command.

## Use COPY instead of ADD

Both the [COPY and ADD](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/#add-or-copy) instructions provide similar functions in a Dockerfile.
COPY is used for copying local files or directories from the Docker host to the image.
ADD can be used for the same thing as well as downloading external files.
Also, if you use a compressed file (tar, gzip, bzip2, etc.) as the `<src>` parameter, ADD will automatically unpack the contents to the given location.
Use COPY unless you really need the ADD functionality, like to add files from an URL or from a compressed file.
COPY is more predictable and less error prone.

## Use metadata labels

You can add labels to your image to help organize images by project, record licensing information, to aid in automation, or for other reasons.
This help users understand how to use the image easily.
The most common label is "maintainer", which specifies the email address and the name of the person maintaining this image.
Labels are set in the Dockerfile using the LABEL command:

```dockerfile
LABEL maintainer="maintainer@address.com"
```

## Use multi-stage builds

[Docker multi-stage builds](https://docs.docker.com/develop/develop-images/multistage-build/) allow you to break up your Dockerfiles into several stages.
For example, you can have a stage for compiling and building your application, which can then be copied to subsequent stages.
Since only the final stage is used to create the image, the dependencies and tools associated with building your application are discarded, leaving a lean and modular production-ready image.
See the Maven/Tomcat example:

```dockerfile
FROM maven AS build
WORKDIR /app
COPY . .
RUN mvn package

FROM tomcat
COPY --from=build /app/target/file.war /usr/local/tomcat/webapps
```

## Use layer caching

In a Dockerfile each command or instruction creates an [image layer](https://docs.docker.com/get-started/09_image_best/#image-layering).
When you rebuild your image, if your Dockerfile hasn't changed, Docker will just use the cached layers to build the image.
Depending on the number of commands and its meaning, the number of packages and the way they are installed, this can take several minutes or even more.
To optimize the [layer caching](https://docs.docker.com/get-started/09_image_best/#layer-caching), you need to know that once a layer changes, all following or downstream layers must be re-created as well.
So, the rule here is to order your commands in the Dockerfile from the least to the most frequently changing commands to take advantage of caching and this way optimize how fast the image gets built.

## Minimize the number of layers

Layers in an image are good but having too many adds complexity and hurts efficiency.
Each layer increases the size of the image since they are cached.
Therefore, as the number of layers increases, the size also increases.
It's a good idea to combine the RUN, COPY, and ADD commands as much as possible since they create layers.
Limit the images you build to about 5-20 layers, including the base image's layers.
Alternatively, you can use multi-stage builds.

## Build one image for all environments

In your CI/CD pipeline, build just one Docker image.
Then, deploy this image into each target environment, using your externalised configuration (environment variables, etc.) to control the behaviour of your app in each environment.
This means that you build an image once and run it in all your environments.
In other words, you deploy for your development, test, or production environment by just changing a configuration.
Building one image per environment is a bit of a container anti-pattern and should really be avoided.

## Use the least privileged user

By default, processes within Docker containers have root privileges that grant them administrative access to both the container and the host.
This opens containers and the underlying host to security vulnerabilities that hackers might exploit.
To avoid these vulnerabilities, set up a least-privileged user that grants only the necessary privileges.
The USER command allows us to manually set the user's ID within the Dockerfile at any point.
Some images already have a generic user bundled in, which you can use.

## Make executables owned by root and not writable

Every executable in a container should be owned by the root user, even if it is executed by a non-root user.
This will block the executing user from modifying existing binaries or scripts, which could enable different attacks.
By following this best practice, you're effectively enforcing container immutability.
To follow this best practice, try to avoid:

```dockerfile
FROM alpine:3.14
RUN adduser -u 1001 -D app
COPY --chown=app:app . /app
USER app
ENTRYPOINT /app/entrypoint.sh
```

Most of the time, you can just drop the `--chown app:app` option (or `RUN chown ...` commands).
The app user only needs execution permissions on the file, not ownership.

## Prevent confidential data leaks

Secrets are sensitive pieces of information such as passwords, database credentials, SSH keys, tokens, and TLS certificates, to name a few.
If you copy them to an intermediate container, they are cached on the layer to which they were added, even if you delete them later.
These should not be baked into your images without being encrypted.
Instead, they should be injected via environment variables, or via build-time arguments, or using an orchestration tool like [Docker Swarm](https://docs.docker.com/engine/swarm/) (via Docker secrets) or [Kubernetes](https://kubernetes.io/) (via Kubernetes secrets).
Also, you can prevent leaking secrets by adding common secret files and folders to your `.dockerignore` file.

## Include health/liveness checks

Docker exposes an API for checking the status of the process running in the container, which provides much more information than just whether the process is "running" or not.
When using plain Docker or Docker Swarm, include a [HEALTHCHECK instruction](https://docs.docker.com/engine/reference/builder/#healthcheck) in your Dockerfile whenever possible.
This is critical for long running or persistent services to ensure they are healthy and manage restarting the service otherwise.
If running your images in Kubernetes, use [livenessProbe configuration](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/) inside the container definitions, as the docker HEALTHCHECK instruction won't be applied.

## Segregate container networks

Docker containers require a network layer to communicate with the outside world through the network interfaces on the host.
The default bridge network exists on all Docker hosts and, if you do not specify a different network, new containers automatically connect to it.
It is strongly recommended not to rely on the default bridge network.
Instead, use custom bridge networks to control which containers can communicate between them.
You can create as many networks as you need and decide which networks each container should connect to.
Ensure that containers can connect to each other only if necessary and avoid connecting sensitive containers to public-facing networks.

## Limit container resources

By default, a container has no resource constraints and can use as much of a given resource as the host's kernel scheduler allows.
It's a good idea to limit the memory and CPU usage of your containers, especially if you're running multiple containers on a single machine.
This can prevent any of the containers from using all available resources and thereby crippling the rest.
Beyond that, when a container is compromised, attackers may try to make use of the underlying host resources to perform malicious activity.
Set Docker [memory and CPU usage limits](https://docs.docker.com/config/containers/resource_constraints/) to minimize the impact of breaches for resource-intensive containers.

## Avoid privileged containers

By default, Docker containers are "unprivileged" and cannot, for example, run a Docker daemon inside a Docker container.
This is because by default a container is not allowed to access any devices, but a "privileged" container is given access to all devices.
You can restrict the application capabilities to the minimal required set using [--cap-drop flag](https://docs.docker.com/engine/reference/run/#runtime-privilege-and-linux-capabilities) in Docker or [securityContext.capabilities.drop](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/#set-capabilities-for-a-container) in Kubernetes.
That way, in case your container is compromised, the range of action available to an attacker is limited.
As a rule, don't run containers using `--privileged` or `--cap-add` flags.

## Set group ownership and file permissions

If a process needs access to files in the container's local file system, the process's user and group should own those files so they are accessible.
In the Dockerfile, set permissions on the directories and files that the process uses.
The root group must own those files and be able to read and write them as needed.
The Dockerfile code looks like the following, where `/some/directory` is the directory with the files that the process needs access to:

```dockerfile
USER 1001
RUN chown -R 1001:0 /some/directory
```

## Run Docker in rootless mode

[Rootless](https://docs.docker.com/engine/security/rootless/) mode allows running the Docker daemon and containers as a non-root user to mitigate potential vulnerabilities in the daemon and the container runtime.
This is extremely important to mitigate vulnerabilities in daemons and container runtimes, which can grant root access of entire nodes and clusters to an attacker.
Rootless mode executes the Docker daemon and containers inside a user namespace.
This is very similar to [userns-remap mode](https://docs.docker.com/engine/security/userns-remap/), except that with `userns-remap mode`, the daemon itself is running with root privileges.
Whereas in rootless mode, both the daemon and the container are running without root privileges.
Rootless mode does not require root privileges even during the installation of the Docker daemon.

## Protect the Docker daemon socket

The [Docker daemon socket](https://docs.docker.com/engine/security/protect-access/) is a Unix network socket that facilitates communication with the Docker API.
By default, this socket is owned by the root user.
If anyone else obtains access to the socket, they will have permissions equivalent to root access to the host.
Allow only trusted users control of the Docker daemon by making sure only trusted users are members of Docker group.
Take note that it is possible to bind the daemon socket to a network interface, making the Docker container available remotely.
To avoid this issue, never make the daemon socket available for remote connections, unless you are using Docker's encrypted HTTPS socket.
Additionally, do not run Docker images with an option like `-v /var/run/docker.sock://var/run/docker.sock`, which exposes the socket in the resulting container.

## Protect the Docker host

Security doesn't just mean protecting the containers themselves, but also the host machines that run them.
Containers on a given host all share that host's kernel.
If an attacker can compromise the host, all your containers are at risk.
This means that using secure, up to date operating systems and kernel versions is vitally important.
Ensure that your patch and update processes are well defined and audit systems for outdated operating system and kernel versions regularly.

## Maintain host isolation

One of the primary concerns when using containers is isolation between the containers and host as well as the isolation among different containers.
[USER namespaces](https://docs.docker.com/engine/security/userns-remap/) are not enabled by default and thus the attacker will be able to modify the files owned by the root user on the host.
This is because root users inside the container will have the same privileges as the root users on the host unless USER namespaces are enabled.
If you enable user namespaces for Docker daemon, it will ensure that the root inside the docker container is running in a separate context that is different from the host's context.

## Filter system calls

Apply system call filters that allow you to choose which calls can be made by containers to the Linux kernel.
In a container, you can choose to allow or deny any system calls.
Not all system calls are required to run a container.
You can monitor the container, obtain a list of all system calls made, explicitly allow those calls and no others.
This approach enables a secure computing mode, thereby reducing possible exposure points to avoid security mishaps, particularly to avert exploitation of Kernel vulnerabilities.
For more details, read the [Seccomp security profiles for Docker](https://docs.docker.com/engine/security/seccomp/) section in the Docker documentation.

## Do not map any ports below 1024

By default, Docker maps container ports to one that's within the 49153–65525 range, but it allows the container to be mapped to a privileged port.
The TCP/IP port numbers below 1024 are considered privileged ports.
Normal users and processes are not allowed to use them for various security reasons.
Don't map any ports below 1024 within a container as they are considered privileged because they transmit sensitive data.
As a rule, ensure only needed ports are open on the container.

## Set PID limits

Each process in the kernel carries a unique PID, and containers leverage Linux PID namespace to provide a separate view of the PID hierarchy for each container.
Putting limits on PIDs effectively limits the number of processes running in each container.
Limiting the number of processes in the container prevents excessive spawning of new processes and potential malicious lateral movement.
Mostly, the benefit here is if your service always runs a specific number of processes, then setting the PID limit to that exact number mitigates many malicious actions, including reverse shells and remote code injection.

## Shutdown smartly and gracefully

In a Dockerized runtime like [Docker Swarm](https://docs.docker.com/engine/swarm/) or [Kubernetes](https://kubernetes.io/), containers are born and die frequently.
This happens not only when errors are thrown but also for good reasons like relocating containers, replacing them with a newer version and more.
It's achieved by sending a notice (SIGTERM signal) to the process with a 10 second grace period.
If the timeout period passes, Docker sends a SIGKILL signal, which causes the process to be stopped by the operating system immediately.
Make sure the app is handling the ongoing requests and clean-up resources in a timely fashion.

## Ensure that containers are stateless and immutable

Containers are designed to be [stateless and immutable](https://cloud.google.com/architecture/best-practices-for-operating-containers#ensure_that_your_containers_are_stateless_and_immutable).
Stateless means that any state (persistent data of any kind) is stored outside of a container.
This external storage can take several forms, depending on what you need.
It allows the container to be cleanly shut down and destroyed at any time without data loss.
If a new container is created to replace the old one, you just connect the new container to the same datastore or bind it to the same disk.
Immutable means that a container won't be modified during its life: no updates, no patches, no configuration changes.
If you must update the application code or apply a patch, you build a new image and redeploy it.
If you need to roll back, you simply redeploy the old image.

## Secure containers at runtime

The ability to stop an attack in progress is of utmost importance but few organizations are effectively able to stop an attack or zero-day exploit as it happens, or before it happens.
Runtime security for Docker containers involves securing your workload, so that once a container is running, drift is not possible, and any malicious action is blocked immediately.
Ideally, this should be done with minimal overhead and rapid response time.
With secure computing ([seccomp](https://docs.docker.com/engine/security/seccomp/)) you can prevent a containerized application from making certain syscalls to the underlying host operating system's kernel.

## Monitor system resources

Excessive resource usage (CPU, memory, network), quick decrease in available disk space, over-average error rate, or increased latency might be signals that something strange is happening in your system.
Collect metrics, like with [Prometheus](https://prometheus.io/).
Configure alerts to quickly get notified when the values exceed the expected thresholds.
Use meaningful dashboards to explore the evolution of metrics, and correlate with changes in other metrics and events happening in your system.

## Monitor container activity

[Container Monitoring](https://aws.amazon.com/cloudwatch/container-monitoring/) is the activity of continuously collecting metrics and tracking the health of containerized applications and microservices environments, in order to improve their health and performance and ensure they are operating smoothly.
Understanding what is happening not just at the cluster or host level, and also within the container runtime and application, helps organizations make better informed decisions.
Creating dashboards will help you visualize these metrics.
[Grafana](https://grafana.com/) is a good choice in many scenarios because of its long list of supported data sources, including [Prometheus](https://prometheus.io/).

## Configure logging mechanisms

As an integral part of application management, logs contain precious information about the events that happen in the application.
Containers offer an easy and standardized way to handle logs because you can write them to `stdout` and `stderr`.
As an application developer, you don't need to implement advanced logging mechanisms.
Most log management systems are time-series databases that store time-indexed documents. You can take advantage of that systems by logging directly in JSON format with different fields.
Some applications can't be easily configured to write logs to `stdout` and `stderr`.
Because such applications write to different log files on disk, the best way to handle them is to use the sidecar pattern for logging.

## Use a secrets management tool

Never store secrets in a Dockerfile that may allow a user with access to the Dockerfile to misplace, misuse, or compromise an entire framework's security.
Standard best practice is to safely encrypt key secrets in third-party tools, such as the [Hashicorp Vault](https://www.vaultproject.io/).
Container orchestrators like Docker Swarm or Kubernetes provide a secrets management capability which can solve this problem.
You can use secrets to manage sensitive data a container needs at runtime, without storing it in the image or in source code.

## Use a private registry

Container registries are highly convenient, letting you download container images at the click of a button, or automatically as part of development and testing workflows.
However, together with this convenience comes a security risk.
There is no guarantee that the image you are pulling from the registry is trusted.
It may unintentionally contain security vulnerabilities or may have intentionally been replaced with an image compromised by attackers.
The solution is to use a private registry deployed behind your own firewall, to reduce the risk of tampering.
[Nexus](https://www.sonatype.com/products/repository-oss) and [Artifactory](https://jfrog.com/artifactory/) are alternatives to [Docker Hub](https://hub.docker.com/).

## Use Docker Compose

[Docker Compose](https://docs.docker.com/compose/) is a tool that was developed to help define and share multi-container applications.
With Compose, you can create a YAML file to define the services and with a single command, can spin everything up or tear it all down.
The big advantage of using Compose is you can define your application stack in a file, keep it at the root of your project repo, and easily enable someone else to contribute to your project.
Also, you don't need to remember command line options to run your containers.

## Use a linter

Linter is a static code analysis tool used to flag programming errors, bugs, stylistic errors, and suspicious code.
Use of a linter to avoid common mistakes and establish best practice guidelines that engineers can follow in an automated way.
This is a helpful docker security scanning task to statically analyse Dockerfile security issues.
One such linter is [hadolint](https://github.com/hadolint/hadolint).
It parses a Dockerfile and shows a warning for any errors that do not match its best practice rules.
Consider incorporating this tool into your CI pipelines.

## Use a build tool

If you use a specialised Docker image build tool, you will benefit from integration with your build tool (e.g. Maven or Gradle) and some opinionated defaults for your apps.
[Jib](https://github.com/GoogleContainerTools/jib) is a tool from Google for building better Docker and [OCI](https://github.com/opencontainers/image-spec) images for Java applications without a Docker daemon.
It helps you to create quite granular builds, because it can separate your Java application into multiple layers.
This means builds can be faster because only the layers that changed need to be rebuilt.

## Use Docker Lock

[Docker Lock](https://github.com/safe-waters/docker-lock) is a cli tool that automates managing image digests by tracking them in a separate Lockfile (think `package-lock.json` or `Pipfile.lock`).
With `docker-lock`, you can refer to images in Dockerfiles, docker-compose files, and Kubernetes manifests by mutable tags (as in python:3.6) yet receive the same benefits as if you had specified immutable digests (as in python:3.6@sha256:25a189a536ae4d7c77dd5d0929da73057b85555d6b6f8a66bfbcc1a7a7de094b).
`docker-lock` is most commonly used as a cli-plugin for Docker.
So, it can be used as subcommand of Docker as in `docker lock`.

## Have an alert system

Alerts allow you to identify problems in the system moments after they occur.
By quickly identifying unintended changes to the system, you can minimize service disruptions.
Alerts consists in alert rules and notification channel.
Configure alerts to quickly get notified when the values exceed the expected thresholds.
Use meaningful dashboards to explore the evolution of metrics, and correlate with changes in other metrics and events happening in your system.

## Bibliography

- [10 best practices to containerize Node.js web applications with Docker](https://snyk.io/blog/10-best-practices-to-containerize-nodejs-web-applications-with-docker/)
- [10 Docker Security Best Practices](https://snyk.io/blog/10-docker-image-security-best-practices/)
- [14 best practices for containerising your Java applications](https://www.tutorialworks.com/docker-java-best-practices/)
- [9 Best practices for Docker Image](https://blog.fibonalabs.com/best-practices-for-docker-image/)
- [Best Practices and Tips for Writing a Dockerfile](https://www.qovery.com/blog/best-practices-and-tips-for-writing-a-dockerfile)
- [Best practices for building containers](https://cloud.google.com/architecture/best-practices-for-building-containers)
- [Best practices for building images](https://developers.redhat.com/articles/2021/11/11/best-practices-building-images-pass-red-hat-container-certification)
- [Best practices for operating containers](https://cloud.google.com/architecture/best-practices-for-operating-containers)
- [Best practices for scanning images](https://docs.docker.com/develop/scan-images/)
- [Best practices for writing Dockerfiles](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)
- [Container Monitoring: Essential Tools + Best Practices](https://scoutapm.com/blog/container-monitoring)
- [Docker and Node.js Best Practices](https://github.com/nodejs/docker-node/blob/main/docs/BestPractices.md)
- [Docker Best Practices for Python Developers](https://testdriven.io/blog/docker-best-practices/)
- [Docker Container Security 101: Risks and 33 Best Practices](https://www.stackrox.io/blog/docker-security-101/)
- [Docker development best practices](https://docs.docker.com/develop/dev-best-practices/)
- [Docker Logging: 8 Best Practices for Data Security](https://cloudlytics.com/docker-logging-best-practices/)
- [Docker Security: 14 Best Practices for Securing Docker Containers](https://www.bmc.com/blogs/docker-security-best-practices/)
- [Dockerfile best practices](https://github.com/hexops/dockerfile)
- [Image-building best practices](https://docs.docker.com/get-started/09_image_best/)
- [Secure Your Containers with this One Weird Trick](https://www.redhat.com/en/blog/secure-your-containers-one-weird-trick)
- [Top 20 Docker Security Best Practices](https://blog.aquasec.com/docker-security-best-practices)
- [Top 20 Dockerfile best practices for security](https://sysdig.com/blog/dockerfile-best-practices/)
- [Top 8 Docker Best Practices for using Docker in Production](https://dev.to/techworld_with_nana/top-8-docker-best-practices-for-using-docker-in-production-1m39)
- [Top Docker Best Practices to Improve Dockerfiles](https://www.datree.io/resources/docker-best-practices)
- [Understanding Container Monitoring: Best Practices and Tools](https://www.aquasec.com/cloud-native-academy/docker-container/container-monitoring/)
