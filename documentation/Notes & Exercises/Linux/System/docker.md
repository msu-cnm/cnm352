## Docker Commands

### Image Management

#### Show all Images

```bash
sudo docker images -a
```

#### Download an image

```bash
sudo docker pull IMAGE_NAME
```

#### Remove an image

```bash
sudo docker rmi IMAGE_ID
```

### Container Management

#### Show containers:

```bash
# Show running containers
sudo docker container ls

# Show all containers
sudo docker ps -a
```

#### Run (create & start) a container:

```bash
sudo docker run -it --entrypoint=/bin/bash --name INSTANCE_NAME -v LOCALDIR:INSTANCE_DIR IMAGE_NAME

# Options
-i	# Keep STDIN open even if not attached
-t	# Allocate a pseudo-tty
--entrypoint=/bin/sh	# Overrides default entrypoint, basically first command it runs
--name INSTANCE_NAME	# The name given to the Docker instance
-v LOCALDIR:INSTANCE_DIR	# Maps a local folder to a folder in the container
IMAGE_NAME	# The Docker image used to make the instance
--rm	# This will automatically delete the container on exit
```

Executing the above command will bring you to a prompt inside your docker container.  

To detach from the container and keep it running (interactive mode to daemon mode)

```bash
CTRL+P,CTRL+Q  # You can just CTRL and hit 'P' then 'Q'
```

To detach and stop the container:

```bash
exit
```

#### Start a stopped container:

This command will start a container that was previously stopped.  It does not interact with the container, it just starts it in the background.

```bash
sudo docker start INSTANCE_NAME
```

#### Interact with a started container:

Brings you to a shell prompt inside the container.  

```bash
sudo docker exec -it INSTANCE_NAME /bin/bash
```

#### Stop a container:

```bash
sudo docker stop INSTANCE_NAME
```

#### Delete Containers

```bash
# Delete a single container
sudo docker rm INSTANCE_NAME

# Delete all stopped containers
sudo docker rm $(sudo docker ps -a -q)

# Force delete a running container
sudo docker rm --force INSTANCE_NAME
```

### Commit Changes to Docker Image

Will be dependent on original image

1. Pull the docker image

2. Deploy a container (use the network switch if you need networking)

   ```bash
   sudo docker run --network host -it 1b3de68a7ff8 /bin/bash
   # Verify network
   ip addr show
   ```

3. Make changes and then exit the container.

4. Find the container ID of it (look for exited time)

   ```bash
   sudo docker ps -a
   ```

5. Commit changes to a new image

   ```bash
   sudo docker commit [CONTAINER_ID] [new_image_name]
   ```

### Creating Custom Image

I didn't do this, but I believe this image self-contained

1. Pull a base image:

   ```bash
   # Ex
   docker pull ubuntu:latest
   docker pull ubuntu:16.04
   ```

2. Create a Dockerfile

   ```bash
   mkdir build-compiler-image
   
   cat <<EOF >build-compiler-image/Dockerfile
   FROM ubuntu:16.04
   RUN apt-get update -y && apt-get install -y gcc
   EOF
   ```

3. Build custom image

   ```bash
   docker build -t build-c build-compiler-image
   ```

### Cheat Sheet

https://www.digitalocean.com/community/tutorials/how-to-remove-docker-images-containers-and-volumes