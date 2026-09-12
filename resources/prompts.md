## Step 1: Generate the Dockerfiles


```
Help me create Dockerfiles for the frontend and backend apps. Frontend will run
on port 3000 and backend on 8080. Keep the docker files separate and do not use
docker compose. Use nginx to serve the frontend files. For the backend, do not
copy the .env file into the Docker image: the env variables will be injected
using a env file separately.
```

## Step 2: Make the frontend API URL configurable

```
make the baseURL in the frontend configurable with VITE_API_BASE_URL.
```

## Step 3: Initialize git & push to GitHub

```
Initialize a git repo, commit the current version and push the changes to a
remote repo https://github.com/<your-username>/<your-repo>
```

## Step 5: Connect to EC2 & Install Docker

```
Connect to the EC2 instance <ec2-public-dns> and install the latest version of
docker on the instance. The ssh key is located in @~/.ssh/<key-file.pem>
```

## Step 6: Clone the repo onto EC2

```
Clone the github repo https://github.com/<your-username>/<your-repo> on the
EC2 instance.
```

## Step 7: Build the two Docker images

```
Build the two docker images on the EC2 instance. 
```

## Step 8: Create the backend `.env` on EC2

```
Use scp to copy the .env file from the backend folder on my local machine to the EC2 instance.
```

## Step 9: Run the two containers

```
Run the two containers on the EC2 instance. I have created a .env file at ~/<your-repo>/threadhive-backend/.env and configured the security group. Run the backend on port 8080 loading variables with --env-file, and the frontend on port 3000. Use --restart unless-stopped for both containers.
```