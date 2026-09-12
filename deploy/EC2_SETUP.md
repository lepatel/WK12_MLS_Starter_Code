# Deploying ThreadHive to EC2 (plain `docker run`, no compose)

Two standalone images, built and run directly on the instance. MongoDB stays on Atlas.

- Backend image listens on **8080**, config injected at runtime via `--env-file` (the `.env` is never baked into the image).
- Frontend image is a static build served by nginx on **3000**; it calls the backend at a URL baked in at build time via `VITE_API_BASE_URL`.

## 1. Security group

| Port | Source        | Purpose  |
| ---- | ------------- | -------- |
| 22   | your IP only  | SSH      |
| 3000 | 0.0.0.0/0     | frontend |
| 8080 | 0.0.0.0/0     | backend (the browser calls it directly, so it must be public) |

## 2. MongoDB Atlas network access

Add the EC2 instance's public IP under Atlas → Network Access (allocate an Elastic IP first if it doesn't have one, so this doesn't break on reboot).

## 3. Connect to EC2 and install Docker

```bash
ssh -i ~/.ssh/<key-file.pem> <user>@<ec2-public-dns>
```

Then, on the instance (Ubuntu):

```bash
sudo apt-get update
sudo apt-get install -y ca-certificates curl gnupg
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo $VERSION_CODENAME) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin

sudo usermod -aG docker $USER
# log out and back in for the group change to take effect
```

## 4. Clone the repo onto EC2

```bash
git clone https://github.com/<your-username>/<your-repo> ~/<your-repo>
cd ~/<your-repo>
```

## 5. Build the two images

```bash
docker build -t threadhive-backend ./threadhive-backend

docker build \
  --build-arg VITE_API_BASE_URL=http://<ec2-public-dns>:8080/api \
  -t threadhive-frontend ./threadhive-frontend
```

`VITE_API_BASE_URL` is compiled into the static JS at build time, so it must be the instance's actual public DNS/IP — not `localhost`.

## 6. Copy the backend `.env` onto EC2

From your local machine (not the instance):

```bash
scp -i ~/.ssh/<key-file.pem> threadhive-backend/.env <user>@<ec2-public-dns>:~/<your-repo>/threadhive-backend/.env
```

Make sure that `.env` has `PORT=8080` (matches `.env.example`) plus `MONGODB_URI`, `JWT_SECRET`, and the Gemini/OpenAI keys.

## 7. Run the two containers

Back on the EC2 instance:

```bash
docker run -d \
  --name threadhive-backend \
  --env-file ~/<your-repo>/threadhive-backend/.env \
  -p 8080:8080 \
  --restart unless-stopped \
  threadhive-backend

docker run -d \
  --name threadhive-frontend \
  -p 3000:3000 \
  --restart unless-stopped \
  threadhive-frontend
```

Verify:

```bash
docker ps
curl http://localhost:8080/api/threads   # or any real backend route
```

Then visit `http://<ec2-public-dns>:3000` in a browser.

## Redeploying after a code change

```bash
cd ~/<your-repo>
git pull
docker build -t threadhive-backend ./threadhive-backend
docker build --build-arg VITE_API_BASE_URL=http://<ec2-public-dns>:8080/api -t threadhive-frontend ./threadhive-frontend
docker stop threadhive-backend threadhive-frontend
docker rm threadhive-backend threadhive-frontend
# then re-run the two `docker run` commands from step 7
```
