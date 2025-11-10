
# 4-6. Creating A Container And Communicating To Web (WWW)

# Step 1: Check .env file
DB_URL=mongodb://localhost:27017/ts-docker-db 

# Step 2: Run the container
docker run -p 5000:5000 --rm --name dev-backend \
  -v ts-backend-logs:/app/logs \
  -v "$(pwd)":/app \
  -v /app/node_modules \
  --env-file .env \
  ts-backend:dev
  
  
# 4-7. Making Container To Host Container Communication

# Step 1: Update .env file with (host.docker.internal)
DB_URL=mongodb://host.docker.internal:27017/ts-docker-db

# STEP 2: Run the container
docker run -p 5000:5000 --rm --name dev-backend \
  --add-host=host.docker.internal:host-gateway \
  -v ts-backend-logs:/app/logs \
  -v "$(pwd)":/app \
  -v /app/node_modules \
  --env-file .env \
  ts-backend:dev
  
  
# 4-8. A Basic Solution For Container To Container Communication

# Step 1: Run container-1 (mongodb-container)
docker run --rm --name mongodb-container mongo

# Step 2: Get mongodb-container's IP address
docker inspect mongodb-container

# Step 3: Update .env file
DB_URL=mongodb://172.17.0.2:27017/ts-docker-db  # Use mongodb-container IP address

# Step 4: Run container-2 (dev-backend)
docker run -p 5000:5000 --rm --name dev-backend \
  -v ts-backend-logs:/app/logs \
  -v "$(pwd)":/app \
  -v /app/node_modules \
  --env-file .env \
  ts-backend:dev


# 4-9. Introduction To Docker Networks
# Docker networks allow containers to communicate directly using container names as hostnames.

# Step 1: Create a custom network
docker network create network-name 

# List networks:
docker network ls
```
```
NETWORK ID     NAME           			DRIVER    SCOPE
abc123def456   bridge         			bridge    local
def456ghi789   host          	 			host      local
ghi789jkl012   none          	 			null      local
jkl012mno345   ts-docker-network   	bridge    local  ← Your custom network

# Step 2: Run Containers on Same Network

# MongoDB on custom network
docker run -d --rm \
  --name mongo-container \
  --network ts-docker-network \
  mongo
  
# Backend on same network

# MUST: Inside .env file
DB_URL=mongodb://mongo-container:27017/ts-docker-db		# ✅ Use container NAME (here, mongodb-container)
  
docker run -p 5000:5000 --rm \
	--name ts-backend-container \
	--network ts-docker-network \
	--env-file .env \
	-v ts-docker-logs:/app/logs \
	-v "$(pwd)":/app \
	-v /app/node_modules \
	ts-backend:dev 
	
	
# Module 5: Building multi container application with Docker
# 5-1. Steps To Build A Multi Container Application
👉 MUST CHECK: 📂 Inside docker/module-5/claude-ai-docker-2: STEPS-TO-FOLLOW.md file
📖 MUST READ: https://claude.ai/share/be32ad91-745c-45e7-8c28-7fb8a17f4736
📖 MUST READ: https://claude.ai/share/b362d7af-eb55-49e5-8421-565eed6482ad

# The Complete Command Summary
# 1. Create network and volume
docker network create myapp-network
docker volume create mongo-data

# 2. Start MongoDB
docker run -d --rm \
  --name mongodb-container \
  --network myapp-network
  -v mongo-data:/data/db \
  -p 27018:27017 \ 👉 Only provide port, when want to connect MongoDB + MongoDB Compass
  mongo

# 3. Build Backend image
docker build -t ts-backend:dev .

# 4. Start Backend
docker run -d --rm \
  -p 5000:5000 \
  --name dev-backend \
  --network myapp-network \
  --env-file .env
  -v ts-docker-logs:/app/logs \
  -v "$(pwd)":/app \
  -v /app/node_modules \
  ts-backend:dev
  
# 5. Start Frontend:
docker run -d --rm \
  -p 3000:3000 \
  --name dev-frontend \
  --network myapp-network \
  --env-file .env \
  -v "$(pwd)":/app \
  -v /app/node_modules \
  nextjs-frontend:dev

# 6. Verify
docker ps -a
docker logs -f dev-backend
docker logs -f dev-frontend
docker logs -f mongodb-container


# Module 6: Docker Compose & Utility Container

# Docker Utility Container
# Step 0: Create a Dockerfile which will be used for building utility container image
# Follow (Step 1 to Step 3) every time

# Step-1 : Build The Utility Container Image
docker build -t node-util .

# Step-2 : Run The Utility Container in Interactive Mode
docker run -d --rm -it --name node-util -v "$(pwd)":/app node-util

# Step-3: Execute Command Inside Utility Container And Install Packages
docker exec -it node-util npm init -y (For Express APP)
docker exec -it node-util npm i express
docker exec -it node-util npm i -D typescript
