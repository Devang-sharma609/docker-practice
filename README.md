# What is Docker?
Docker is a containerization platform.
It lets developers package applications along with all dependencies (like OS libraries, frameworks, runtime) into containers so they run identically anywhere — on your laptop or on the cloud.

## 💡 Why Docker?

Traditional deployment (VMs, manual setups) had issues like:

- “Works on my machine” problems
- Heavy virtual machines (VMs) with separate OS
- Complex dependency management

Docker solves this by:

- Creating **isolated environments** using Linux kernel features (namespaces + cgroups(control groups))
- Sharing the host OS kernel (so it’s super lightweight)
- Running multiple apps on the same machine without conflict

## ⚙️ Core Docker Concepts

| Concept | Explanation |
| --- | --- |
| **Image** | Blueprint for your container (like a template). Built from Dockerfile. |
| **Container** | Running instance of an image. Think: image = class, container = object. |
| **Dockerfile** | Text file with instructions to build an image. |
| **Docker Hub** | Public registry where images are stored (like GitHub for containers). |
| **Docker Engine** | Core runtime that builds and runs containers. |
| **Volumes** | Way to persist data beyond the life of a container. |
| **Networks** | Allows containers to talk to each other securely. |

## 🚀 Basic Docker Commands

| Command | Description |
| --- | --- |
| `docker build -t myapp .` | Build an image from Dockerfile |
| `docker images` | List all images |
| `docker run -p 3000:3000 myapp` | Run a container and map port 3000 |
| `docker ps` | Show running containers |
| `docker stop <container_id>` | Stop a container |
| `docker rm <container_id>` | Remove container |
| `docker rmi <image_id>` | Remove image |
| `docker exec -it <container_id> bash` | Enter into a running container |
| `docker logs <container_id>` | Check container logs |
| `docker-compose up` | Run multi-container app from docker-compose.yml |

### Dockerfile

```dockerfile
FROM maven:3.9.6-eclipse-temurin-17 AS build
# "build" is the reference name of this stage which'll be used later to copy artifacts

# Set working directory inside the container
WORKDIR /app

# Copy only pom.xml first to leverage Docker layer caching
COPY pom.xml .

# Download dependencies (cached if pom.xml unchanged)
RUN mvn dependency:go-offline -B

# Copy all source code
COPY src ./src

# Package the application (skip tests for faster build)
RUN mvn clean package -DskipTests

# ---------- Stage 2: Runtime ----------
# Use lightweight JDK runtime image
FROM eclipse-temurin:17-jdk-alpine

# Set working directory inside runtime container
WORKDIR /app

# Copy the compiled jar from the build stage
COPY --from=build /app/target/*.jar app.jar

# Expose Spring Boot default port
EXPOSE 8080

# Run the Spring Boot application
# this command cant be changed, to enable customization, use CMD ["java","-jar","app.jar"]
ENTRYPOINT ["java","-jar","app.jar"]
```

### How multi-stage Dockerfile saves time?

Docker caches everything for reuse in case of no changes.

so if us deps remain unchanged, dep. installation stage is skipped and moves to app build stage(if it has changes).

### Docker-compose

```yaml
# The 'services' section defines all the containers (services)
# that will run together in this multi-container setup
services:
  # ---------- BACKEND SERVICE ----------
  backend:
    build: ./backend # Build the backend image from Dockerfile located in ./backend
    
    ports: # Map the host machine's port 8080 to the container's port 8080
      - "8080:8080" # This allows access to the backend at http://localhost:8080

    environment: # Define environment variables that the backend needs
      DB_HOST: db                 # The hostname is the service name 'db'
      DB_PORT: 5432               # Default PostgreSQL port
      DB_USER: devang             # Must match the POSTGRES_USER in db service
      DB_PASSWORD: secret         # Must match the POSTGRES_PASSWORD in db service
      DB_NAME: myappdb            # Database name backend will use

    
    depends_on: # Ensure backend starts after db is ready
      - db

  # ---------- FRONTEND SERVICE ----------
  frontend:
    build: ./frontend # Builds the frontend image from Dockerfile at -- ./frontend

    ports:
      - "3000:3000" # Map port 3000 on host to port 3000 inside container

    environment:
      REACT_APP_API_URL: http://backend:8080

    # Ensure frontend waits for backend to start
    depends_on:
      - backend

  # ---------- DATABASE SERVICE ----------
  db:
    image: postgres:16 # Pulls the official PostgreSQL image from Docker Hub
    ports:
      - "5432:5432" # Exposes port -"hostPort":"containerPort"
      # You can access the DB externally with localhost:5432 if needed

    environment: # Sets required environment variables for DB
      POSTGRES_USER: devang        # Creates database user 'devang'
      POSTGRES_PASSWORD: secret    # Sets password for the user
      POSTGRES_DB: myappdb         # Creates a database named 'myappdb'

    # Define volume for persistent data storage
    # Data written inside /var/lib/postgresql/data will survive restarts on your host machine (outside the container)
    volumes:
      - db_data:/var/lib/postgresql/data

# ---------- NAMED VOLUMES ----------
# This section defines the named volume referenced above.
# Docker persists it ,
volumes:
  db_data:
```

Use `.dockerignore` to avoid copying unnecessary files to your docker container

## 📝 Official References

### Docker Basics
- [Docker Overview](https://docs.docker.com/get-started/overview/)
- [Docker Engine](https://docs.docker.com/engine/)

### Dockerfile
- [Dockerfile reference](https://docs.docker.com/engine/reference/builder/)
- [Multi-stage builds](https://docs.docker.com/develop/develop-images/multistage-build/)

### Docker CLI Commands
- [docker build](https://docs.docker.com/engine/reference/commandline/build/)
- [docker run](https://docs.docker.com/engine/reference/commandline/run/)
- [docker ps](https://docs.docker.com/engine/reference/commandline/ps/)
- [docker exec](https://docs.docker.com/engine/reference/commandline/exec/)
- [docker logs](https://docs.docker.com/engine/reference/commandline/logs/)
- [docker stop](https://docs.docker.com/engine/reference/commandline/stop/)
- [docker rm](https://docs.docker.com/engine/reference/commandline/rm/)
- [docker rmi](https://docs.docker.com/engine/reference/commandline/rmi/)
- [docker inspect](https://docs.docker.com/engine/reference/commandline/inspect/)
- [docker image prune](https://docs.docker.com/engine/reference/commandline/image_prune/)

### Volumes
- [Docker Volumes](https://docs.docker.com/storage/volumes/)

### Networking
- [Docker Networks](https://docs.docker.com/network/)
- [Bridge network overview](https://docs.docker.com/network/bridge/)

### Docker Compose
- [Docker Compose Overview](https://docs.docker.com/compose/)
- [docker-compose up](https://docs.docker.com/compose/reference/up/)
- [docker-compose ps](https://docs.docker.com/compose/reference/ps/)
- [docker-compose logs](https://docs.docker.com/compose/reference/logs/)
- [docker-compose exec](https://docs.docker.com/compose/reference/exec/)
- [docker-compose down](https://docs.docker.com/compose/reference/down/)

### Git & GitHub
- [GitHub Quickstart](https://docs.github.com/en/get-started/quickstart)
- [Git Basics](https://git-scm.com/book/en/v2/Getting-Started-Git-Basics)

## Docker Practice Roadmap

#### Setup
- [ ] clone repo
- [ ] install docker and docker-compose

#### Backend
- [ ] move backend app to docker
- [ ] add a Dockerfile for backend
- [ ] build your backend image
- [ ] run backend container
- [ ] While it’s running, cd into your frontend (unable to do so....?)
- [ ] Fix it by running backend in detached mode

#### Frontend
- [ ] move frontend app to docker
- [ ] add a Dockerfile for frontend
- [ ] build your frontend image
- [ ] run frontend container

#### Optimization
- [ ] setup multi-stage builds for both frontend and backend
- [ ] use .dockerignore to exclude unnecessary files

#### Important
- [ ] List running containers
- [ ] Find container ID / hash of running backend
- [ ] Attach to running backend
- [ ] Explore /app inside container
- [ ] Check logs
- [ ] Follow logs in real-time

#### Stop & Remove Containers / Images
- [ ] Stop backend container
- [ ] Remove backend container
- [ ] Remove backend image

#### Volumes & Data Persistence
- [ ] Run backend with volume
- [ ] Create a file inside /app/logs in container
- [ ] Stop and remove container, then run again
- [ ] Verify data persists on host

#### Docker Compose
- [ ] Create docker-compose.yml with backend, frontend, and db
- [ ] Run all services in detached mode: docker-compose up -d
- [ ] List services: docker-compose ps
- [ ] Check logs: docker-compose logs backend
- [ ] Attach to backend container: docker-compose exec backend bash
- [ ] Rebuild after code change: docker-compose up -d --build
- [ ] Stop all services

## 📢 Contributing to this Repo

Welcome! This repository is designed as a **hands-on Docker practice repo**. Contributions from other developers are highly encouraged! If you are visiting this repo and want to practice, here’s how you can contribute safely and submit your code as example for other devs working with your tech stack while learning.

#### 1️⃣ Fork the Repository
- Click the **Fork** button in the top-right corner of this repo.  
- This creates a copy of the repository under your GitHub account.  
- You can freely experiment and create branches in your fork.

#### 2️⃣ Clone Your Fork Locally
```bash
git clone https://github.com/Devang-sharma609/docker-practice
cd docker-practice
```

#### 3️⃣ Create a New Branch
```bash
git checkout -b <your-tech-stack>
```
For eg: git checkout -b kotlin

#### 4️⃣ Make Changes & Commit
Keep your commit messages clear and descriptive.
```bash
git add .
git commit -m "Completed Docker practice tasks for Kotlin"
git push origin <your-tech-stack>
```

#### 5️⃣ Push Branch to Your Fork
```bash
git push origin <your-tech-stack>
```

#### 6️⃣ Create a Pull Request
1. Go to your fork on GitHub.
2. Click Compare & pull request next to your branch.
3. Set the base branch of the original repo to mern, NOT main.
4. Add a title and description explaining your changes.
5. Click Create pull request.⚠️ All accepted PRs will be merged into the separate branches. This keeps main stable while allowing contributors to add code and examples safely.