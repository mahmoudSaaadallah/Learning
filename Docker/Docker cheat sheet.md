Here is a handy Docker cheat sheet categorized by what you are trying to do. It covers the most common commands you will use daily.

### 🐳 1. Working with Images (The Recipes)

Images are the blueprints for your containers.

|**Command**|**What it does**|
|---|---|
|`docker pull <image_name>`|Downloads an image from Docker Hub (e.g., `docker pull ubuntu`).|
|`docker build -t <name> .`|Builds an image from the `Dockerfile` in your current folder (`.`) and tags it with a name.|
|`docker images`|Lists all the images currently downloaded/built on your computer.|
|`docker rmi <image_name>`|Deletes a specific image from your computer.|
|`docker image prune`|Cleans up and deletes any unused, unnamed "dangling" images to free up space.|

---

### 📦 2. Working with Containers (The Running Apps)

Containers are the actual running instances of your images.

|**Command**|**What it does**|
|---|---|
|`docker run <image_name>`|Creates and starts a new container from an image.|
|`docker ps`|Lists only your **currently running** containers.|
|`docker ps -a`|Lists **all** containers (both running and stopped).|
|`docker start <container_name>`|Starts a container that was previously stopped.|
|`docker stop <container_name>`|Gracefully stops a running container.|
|`docker restart <container_name>`|Stops and then immediately starts a container.|
|`docker rm <container_name>`|Deletes a **stopped** container.|
|`docker rm -f <container_name>`|Force-deletes a container even if it is currently running.|

#### ✨ Crucial `docker run` Flags:

When starting a container, you almost always use these extra settings:

- **`-d`** (Detach): Runs the container in the background so you can keep using your terminal.
    
- **`-p 8080:80`** (Publish): Maps your computer's port (8080) to the container's port (80).
    
- **`--name my_app`**: Gives your container a readable name instead of a random string.
    
- _Example:_ `docker run -d -p 8080:80 --name web_server nginx`
    

---

### 🛠️ 3. Troubleshooting & Inspecting

When things go wrong, these are the commands you use to figure out why.

|**Command**|**What it does**|
|---|---|
|`docker logs <container_name>`|Shows the text output/errors from your application.|
|`docker logs -f <container_name>`|Same as above, but **follows** the logs live (like watching a TV screen).|
|`docker exec -it <container> sh`|Opens a terminal **inside** the running container so you can look at files. (Type `exit` to leave).|
|`docker inspect <container_name>`|Spits out all the raw details (IP addresses, configurations) of a container in JSON format.|
|`docker top <container_name>`|Shows you the active processes (like Task Manager) running inside the container.|

---

### 🧹 4. System Cleanup (The Housekeeping)

Docker can eat up your hard drive space quickly over time. Use these to clean up.

|**Command**|**What it does**|
|---|---|
|`docker container prune`|Deletes **all** stopped containers at once.|
|`docker system prune`|**The Nuke:** Deletes all stopped containers, unused networks, dangling images, and build cache. (Highly recommended to run occasionally).|
|`docker system prune -a`|**The Bigger Nuke:** Same as above, but deletes _every_ image that isn't currently being used by a running container.|
|`docker system df`|Shows you exactly how much disk space Docker is currently using.|

_(Tip: Keep this saved somewhere accessible. As you practice, these commands will quickly become second nature!)_