In this DevOps task, you need to build and deploy a full-stack CRUD application using the MEAN stack (MongoDB, Express, Angular 15, and Node.js). The backend will be developed with Node.js and Express to provide REST APIs, connecting to a MongoDB database. The frontend will be an Angular application utilizing HTTPClient for communication.  

The application will manage a collection of tutorials, where each tutorial includes an ID, title, description, and published status. Users will be able to create, retrieve, update, and delete tutorials. Additionally, a search box will allow users to find tutorials by title.

## Project setup

### Node.js Server

cd backend

npm install

You can update the MongoDB credentials by modifying the `db.config.js` file located in `app/config/`.

Run `node server.js`

### Angular Client

cd frontend

npm install

Run `ng serve --port 8081`

You can modify the `src/app/services/tutorial.service.ts` file to adjust how the frontend interacts with the backend.

Navigate to `http://localhost:8081/`

# 🎯 CRUD MEAN Stack Application — CI/CD with Jenkins, Docker, and Nginx

This project is a MEAN (MongoDB, Express.js, Angular, Node.js) based CRUD application deployed using a complete CI/CD pipeline.
The pipeline builds Docker images, pushes them to Docker Hub, and deploys the latest version automatically on the application server.

### This README includes:

✔️ Architecture overview

✔️ Step-by-step deployment instructions

✔️ Full CI/CD pipeline with Jenkins

✔️ Docker & Docker Compose setup

✔️ Nginx reverse proxy setup

✔️ Webhook integration

✔️ Screenshot placeholders (for interviewer)

## 🏗️ 1. Architecture Overview

                   ┌────────────────────────┐
                   │     GitHub Repo         │
                   │ crud-dd-task-mean-app   │
                   └───────────┬────────────┘
                               │ Webhook
                               ▼
                   ┌────────────────────────┐
                   │   Jenkins CI Server     │
                   │ (Build + Push images)   │
                   └───────────┬────────────┘
                               │ SSH Deploy
                               ▼
                  ┌──────────────────────────┐
                  │     App Server (EC2)     │
                  │ Docker + Docker Compose  │
                  │ Nginx Reverse Proxy      │
                  └───────────┬─────────────┘
                              ▼
                   http://44.200.190.21

## 📂 2. Repository Structure

```
crud-dd-task-mean-app/
│
├── backend/                  # Node.js API
│   ├── Dockerfile
│   ├── app/
│   └── package.json
│
├── frontend/                 # Angular UI
│   ├── Dockerfile
│   └── package.json
│
├── docker-compose.yml        # Multi-container deployment (MongoDB + Backend + Frontend)
├── Jenkinsfile               # CI/CD pipeline used by Jenkins
└── README.md                 # Complete documentation
```
## ⚙️ 3. Requirements
### A. Jenkins Server (CI Server)

- Ubuntu EC2 instance
- Jenkins installed
- Docker installed
- Docker Hub credentials added in Jenkins
- SSH key added for App server login
- GitHub Webhook configured
  
  <img width="959" height="450" alt="image" src="https://github.com/user-attachments/assets/ec5cb1ef-36b1-431e-84c9-a7ce175b9251" />
  
  <img width="959" height="472" alt="image" src="https://github.com/user-attachments/assets/c40ed529-50b7-4390-a562-094658815eba" />

### B. App Server (Deployment Server)

- Ubuntu EC2 instance
- Docker & Docker Compose installed
- Nginx installed
- /opt/mean-app created for deployment
- 
<img width="959" height="473" alt="image" src="https://github.com/user-attachments/assets/d544ada1-587b-4285-b843-78d8fe3010dc" />

<img width="959" height="472" alt="image" src="https://github.com/user-attachments/assets/ea59c0f2-6ff6-4eca-928e-24e21db65544" />

  
## 🖥️ 4. App Server Setup (Ubuntu EC2)

(must perform these steps on the app server, NOT Jenkins server.)

### 1️⃣ Install Docker

```bash
sudo apt update
sudo apt install -y docker.io
sudo systemctl enable docker
sudo systemctl start docker
sudo usermod -aG docker $USER
```
Logout & login again.

### 2️⃣ Install Docker Compose

```bash
sudo apt install docker-compose -y
```
### 3️⃣ Create project directory
```bash
sudo mkdir -p /opt/mean-app
sudo chown $USER:$USER /opt/mean-app
cd /opt/mean-app
```

<img width="959" height="472" alt="image" src="https://github.com/user-attachments/assets/ea59c0f2-6ff6-4eca-928e-24e21db65544" />


### 4️⃣ Clone project
```bash
git clone https://github.com/DeepikaCAshok/crud-dd-task-mean-app.git
```

<img width="959" height="472" alt="image" src="https://github.com/user-attachments/assets/eb2ac7cd-fe24-4bd0-8495-fad8ea12c356" />


### 5️⃣ Run using Docker Compose
```bash
cd crud-dd-task-mean-app
docker-compose up -d
```
<img width="959" height="472" alt="image" src="https://github.com/user-attachments/assets/5bddac25-c351-40be-900a-69b256973073" />

**Open in browser:**

👉 **Frontend:** [http://44.200.190.21](http://44.200.190.21)

<img width="958" height="473" alt="image" src="https://github.com/user-attachments/assets/06cef6db-cb64-466e-bfd9-ef2b09d9752e" />

👉 **Backend API:** [http://44.200.190.21/api](http://44.200.190.21/api)

<img width="958" height="476" alt="image" src="https://github.com/user-attachments/assets/bf9c0867-fa03-4191-854c-e254128ae492" />


## 🌐 5. Nginx Reverse Proxy Setup (App Server)

**Install Nginx**

```bash
sudo apt install nginx -y
```
**Create config**

```bash
sudo nano /etc/nginx/sites-available/mean-app
```
Code:

```bash
server {
    listen 80 default_server;
    server_name _;

    # Frontend (Angular on host port 4200)
    location / {
        proxy_pass http://127.0.0.1:4200;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }

    # Backend API (Node on host port 3000)
    location /api/ {
        proxy_pass http://127.0.0.1:3000/;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
  ```

<img width="950" height="470" alt="image" src="https://github.com/user-attachments/assets/23b63861-4b08-420d-9bc1-600a23231064" />

**Enable config**

```bash
sudo ln -s /etc/nginx/sites-available/mean-app /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
 ```

<img width="946" height="474" alt="image" src="https://github.com/user-attachments/assets/bd40ab65-bd31-4a48-ad58-9a31f2c70a07" />

## 🔄 6. CI/CD Pipeline (Jenkins)
✔️ **Jenkinsfile included in repo:**

**The pipeline automatically:**

1️⃣ Checks out GitHub code

2️⃣ Builds Backend Docker image

3️⃣ Builds Frontend Docker image

4️⃣ Pushes both images to Docker Hub

5️⃣ SSH into App Server

6️⃣ Pulls latest images

7️⃣ Restarts containers using Docker Compose

<img width="959" height="470" alt="image" src="https://github.com/user-attachments/assets/449185f6-4ed5-4b53-ba94-a1568dc15ad3" />

<img width="959" height="478" alt="image" src="https://github.com/user-attachments/assets/f1df28c7-bf11-4ea3-a346-f3301b81f5c2" />

<img width="959" height="466" alt="image" src="https://github.com/user-attachments/assets/3a7cf8f7-002b-4dc7-92ed-5ed4c4d69e32" />

<img width="959" height="473" alt="image" src="https://github.com/user-attachments/assets/7268326f-8b3e-46a8-b296-102669607c2b" />

<img width="957" height="473" alt="image" src="https://github.com/user-attachments/assets/b5f39329-222b-4075-b760-2911a22ebcc4" />

<img width="947" height="466" alt="image" src="https://github.com/user-attachments/assets/c7454d11-1201-4b71-b19d-e358dac35fb7" />


## 🔧 7. Jenkins Configuration

### A. Add Docker Hub Credentials

Navigate in Jenkins:
```
Jenkins → Manage Jenkins → Manage Credentials → Global → Add Credentials
```
Add:
```
ID: dockerhub
Username: YOUR_DOCKERHUB_USERNAME
Password: YOUR_DOCKERHUB_PASSWORD
```

<img width="959" height="475" alt="image" src="https://github.com/user-attachments/assets/77550282-d19d-4f27-a1af-dfae05768599" />

### B. Add SSH Key Credentials (for App Server Deployment)
Navigate:
```
Jenkins → Manage Jenkins → Manage Credentials → Global → Add Credentials
```
Choose:
```
Kind: SSH Username with private key
ID: app-ec2-ssh
Username: ubuntu
Private Key: (Paste your EC2 .pem file content)
```

<img width="959" height="472" alt="image" src="https://github.com/user-attachments/assets/8d9156e5-cff7-4ae1-85bc-133d630f770e" />

### C. Enable Webhook Trigger for CI/CD
Go to your Jenkins Pipeline Job:
```
Job → Configure → Build Triggers
```
Check:
```
✔️ GitHub hook trigger for GITScm polling

```
<img width="959" height="474" alt="image" src="https://github.com/user-attachments/assets/c05dff6f-aa78-4e81-a50a-fe95b1d6807c" />

<img width="957" height="473" alt="image" src="https://github.com/user-attachments/assets/0f0b65b8-f5ca-4322-b3ff-caa812245131" />

<img width="959" height="470" alt="image" src="https://github.com/user-attachments/assets/9637ad7e-e89d-4231-8cbd-aebd5c66c2f4" />


### 🌐 8. Configure GitHub Webhook
Navigate in GitHub:
```
GitHub → Repository → Settings → Webhooks   → Add Webhook
```
Configure the webhook:
```
Payload URL: http://<JENKINS_PUBLIC_IP>/github-webhook/
Content Type: application/json
Triggers: Just the push event
```

<img width="955" height="473" alt="image" src="https://github.com/user-attachments/assets/21eef4d1-2058-42e0-8489-a822901977b8" />

## 🚀 9. Jenkinsfile Overview

**pipeline performs:**

✔ Build Backend Docker image

✔ Build Frontend Docker image

✔ Login to Docker Hub

✔ Push images

✔ SSH deploy to App Server

✔ Pull latest

✔ Bring up containers

## 🧪 10. Test the Application

**Everything is fully automated.**

### Open in browser:

👉 **Angular UI:**  
[http://44.200.190.21](http://44.200.190.21)

<img width="958" height="473" alt="image" src="https://github.com/user-attachments/assets/2524e961-0596-4750-a912-056aef6e339b" />


### API Test:

👉 **Backend API:**  
[http://44.200.190.21/api/](http://44.200.190.21/api/)

<img width="958" height="476" alt="image" src="https://github.com/user-attachments/assets/2ab0bc9d-f3c4-408a-be96-17390b7b87b5" />

## 🎉 11. Assignment Status

**All required tasks implemented:**

✔ Dockerized MEAN app

✔ Docker Compose deployment

✔ Jenkins CI/CD pipeline

✔ Docker Hub image push

✔ Auto deployment to VM

✔ Nginx reverse proxy

✔ Webhook-based automation

✔ Professional README

**Author**

**Deepika C A**

**Docker | Jenkins | CI/CD | Cloud Deployment**



