# Docker-Image-with-Non-Root-User
Building a Docker Image with Non-Root User Permissions: A Step-by-Step Guide

This guide focuses on creating a secure Docker image using non-root user permissions during the image build process. Running services in Docker containers as a non-root user is a best practice, as it enhances security by limiting the scope of potential damage if the container is compromised. In this guide, we will walk through building a Docker image with proper user management, downloading content from GitHub, setting permissions, and serving content via Apache.

---

## Dockerfile Breakdown

The Dockerfile defines the instructions to create the Docker image. Below are the key concepts:

### Base Image:
- `FROM ubuntu`: Specifies that the Docker image will use Ubuntu as its base.

### Installing Packages:
- `RUN apt update && apt install apache2 unzip -y`: Updates the Ubuntu package list, installs the Apache2 web server, and installs the unzip utility.

### Labels:
- `LABEL maintained-by=Vish` and `LABEL vendor=vish`: These labels provide metadata about the image, useful for tracking and management.

### Environment Variable:
- `ENV var=testing`: Sets the environment variable `var` to "testing", which will be used later in the Dockerfile.

### Setting the Working Directory:
- `WORKDIR /var/www/html/`: Sets the working directory to `/var/www/html/`, the default directory where Apache serves its web content.

### Downloading Content:
- `ADD https://github.com/sharmarvellous/testing/archive/refs/heads/main.zip /var/www/html/code.zip`: Downloads a ZIP file from the specified GitHub repository and saves it as `code.zip` in the working directory.

### Unzipping and Moving Files:
- `RUN unzip code.zip && mv /var/www/html/testing-main/* /var/www/html/`: Extracts the ZIP file and moves the contents to `/var/www/html/`.

### Adding a Non-Root User:
- `RUN adduser nonroot && chown nonroot:nonroot /var/www/html/ -R`: Creates a non-root user named `nonroot` and changes the ownership of the `/var/www/html/` directory to the non-root user.

### Switching to Non-Root User:
- `USER nonroot`: Switches the user from `root` to `nonroot`, ensuring that subsequent operations are performed with limited privileges.

### Cleaning Up:
- `USER root`: Switches back to the root user to perform administrative tasks.
- `RUN rm code.zip`: Removes the `code.zip` file to save space after the files are extracted.

### Starting Apache:
- `CMD ["apache2ctl", "-D", "FOREGROUND"]`: Starts the Apache web server in the foreground, ensuring the container stays active and serves the website.

---

## Image Build Process

The Docker image can be built using the following command:

```bash
docker build -t testing:v1 -f dockerfile7 .
```

## Build Process:

Each step in the Dockerfile is executed sequentially, creating layers for each instruction. This includes downloading the ZIP file from GitHub, extracting it, setting up the non-root user, and preparing Apache to serve the content.

---

## Running the Container
Once the image is built, the container can be run with the following command:

### Explanation:

- `docker build`: Builds a Docker image from the Dockerfile.
- `-t testing:v1`: Tags the image as `testing:v1`, making it easier to reference later.
- `-f dockerfile7`: Specifies `dockerfile7` as the Dockerfile to use for building the image.

```
docker run -d -p 9090:80 --name testc testing:v1
```

### Explanation:
- `docker run`: Starts a new container from the testing:v1 image.
- `-d`: Runs the container in detached mode (background).
- `-p 9090:80`: Maps port `9090` on your local machine to port `80` in the container, where Apache is running.
- `--name testc`: Assigns the name testc to the container.

### Outcome:
The container starts running, and Apache serves the content at `http://localhost:9090`. You can now access the web page, which includes the files that were downloaded and extracted from the GitHub repository.

---

## Verifying the Files in the Running Container

After starting the container, you can inspect the files inside it by executing:

```
docker exec -it testc bash
```

### Explanation:
- `docker exec`: Allows you to execute commands inside a running container.
- `-it testc bash`: Opens an interactive shell inside the testc container.

### Checking the Files:
Once inside the container, you can list the files in the `/var/www/html/` directory with:

```
ls /var/www/html/
```

The expected output should show the files extracted from the ZIP file:
```
README.md  index.html  scripts.js  styles.css  testing-main
```
