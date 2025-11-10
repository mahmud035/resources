# Essential Docker Commands You'll Use Daily:

# Images
docker images                    # List images
docker pull <image-name>         # Download an image
docker rmi <image-ids>           # Remove one or more images
docker image prune               # Remove dangling images
docker image prune -a            # Remove all unused images
docker build -t <image-name>:<tag> pathToDockerfile		# Build an image from a Dockerfile

# Container life-cycle
docker ps                         # Show running containers
docker ps -a                      # Show all containers
docker stop <container-name>      # Graceful stop
docker start <container-name>     # Start stopped container
docker restart <container-name>   # Restart
docker rm <container-name>        # Remove (must be stopped first)
docker rm -f <container-name>     # Force remove
docker run -d -p 5000:5000 --rm --name container-name image:tag    # Create and run a new container from an image

# Here,
-d  --detach		Run container in background and print container ID
-p  --publish   Publish a containers port(s) to the host
    --rm        Automatically remove the container and its associated anonymous volumes when it exits
    --name 			Assign a name to the container
-v, --volume    Bind mount a volume
    --mount     Attach a filesystem mount to the container
    --env-file  Read in a file of environment variables
    

# Volumes and Bind Mounts
docker volume create <volume-name>		# Create a volume
docker volume ls									    # List volumes
docker volume rm <volume-name>			  # Remove a Volume

docker run -v volume-name:/container/path image:tag		# Start a container with a Named volume from an image
docker run --mount type=volume,source=volume-name,target=/container/path		# Another syntax for Named volume

docker run --mount type=bind,source=/host/path,target=/container/path image:tag		# Start a container with a Bind Mount from an image
docker run -v "$(pwd)":/app image:tag		# Shorter syntax for Bind Mount

# Full example using `--mount`: More verbose but clearer, especially in complex setups.
docker run -d \
  -p 5000:5000 --rm \
  --name dev-mount-syntax \
  --mount type=bind,source="$(pwd)",target=/app \
  --mount type=volume,target=/app/node_modules \
  bind-demo:dev
  
# Same using `-v`:
docker run -d \
  -p 5000:5000 --rm \
  --name dev-mount-syntax \
  -v "$(pwd)":/app \
  -v /app/node_modules \
  bind-demo:dev

# Logs and debugging
docker logs <container-name>          	# View logs
docker logs -f <container-name>       	# Follow Logs (like tail -f)
docker exec -it <container-name> sh   	# Shell into container
docker inspect <container-name>       	# Detailed info

# Networking
docker network create network-name		                                  # Create a network
docker network ls		                                                    # List networks
docker run -d --name container-name --network network-name image:tag		# Run container on network
docker network connect network-name container-name		                  # Connect existing container
docker network disconnect network-name container-name	                  # Disconnect
docker network inspect network-name	                                  	# Inspect network
docker network rm network-name		                                      # Remove network


# Cleanup
docker system prune              	# Remove unused data
docker system prune -a           	# Remove EVERYTHING unused
docker volume prune             	# Remove unused volumes
docker image prune                # Remove unused images
docker network prune              # Remove unused networks

# System-wide Docker disk usage
docker system df

# Detailed breakdown
docker system df -v
