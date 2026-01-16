Project: Deploy Multi-App Web Applications on AWS EC2 with Docker, NGINX, and Load Balancer

This document provides a step-by-step, flow-wise guide to deploy two web applications (Bank and Mobile) on AWS EC2, using Docker, Docker Compose, and NGINX as a reverse proxy/load balancer.

1. Prerequisites

AWS Account with access to EC2

EC2 Instance (Amazon Linux 2023 recommended)

Security group with SSH (22) open

Local machine with terminal/SSH access

GitHub account with project source code

Basic knowledge of Docker, Docker Compose, and NGINX

2. Project Structure
   
project-root/
│
├── bank/

│   ├── Dockerfile

│   └── index.html
│
├── mobile/

│   ├── Dockerfile

│   └── index.html
│
├── nginx/

│   ├── Dockerfile

│   └── nginx.conf
│
└── docker-compose.yml

bank/ → Bank app static HTML

mobile/ → Mobile app static HTML

nginx/ → Reverse proxy/load balancer configuration

docker-compose.yml → Multi-container orchestration

3. Step-by-Step Deployment Flow
Step 1: Connect to EC2 Instance
ssh -i <your-key>.pem ec2-user@<EC2-Public-IP>
Step 2: Update EC2 and Install Docker
sudo yum update -y
sudo amazon-linux-extras enable docker
<img width="1920" height="1080" alt="Screenshot (695)" src="https://github.com/user-attachments/assets/fe34a373-62c0-4798-94e9-ef77e00565ec" />
sudo yum install docker -y
sudo systemctl start docker
sudo systemctl enable docker

Check Docker version:

docker --version
Step 3: Install Docker Compose (Standalone)
<img width="1920" height="1080" alt="Screenshot (698)" src="https://github.com/user-attachments/assets/825ae6a7-8af2-47a4-8fdf-045a70aacc35" />

sudo curl -L "https://github.com/docker/compose/releases/download/v2.21.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose

Check version:

docker-compose --version
Step 4: Clone Your Project
<img width="1920" height="1080" alt="Screenshot (696)" src="https://github.com/user-attachments/assets/d0035ad1-68a2-499f-a357-8ed046585fb5" />
git clone <your-github-repo-url>
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/21b6670f-ee67-4eda-b06c-01f8a3e7a79f" />

cd <repo-folder>
Step 5: Edit NGINX Config (Reverse Proxy / Load Balancer)

Edit nginx/nginx.conf using nano:
<img width="1920" height="1080" alt="Screenshot (699)" src="https://github.com/user-attachments/assets/f6f1b807-6e17-46fa-bbe6-77daad86106b" />

sudo nano nginx/nginx.conf

Sample config (Docker-friendly):

upstream loadbalancer {
    server mobile-app:80 weight=6;
    server bank-app:80 weight=4;
}


server {
    listen 80;
    location / {
        proxy_pass http://loadbalancer;
    }
}

Save and exit:

Press CTRL + O → ENTER

Press CTRL + X

Step 6: Check Dockerfiles and docker-compose.yml
<img width="1920" height="1080" alt="Screenshot (700)" src="https://github.com/user-attachments/assets/c0b44b69-ef86-4971-8a86-858e36b4fb6f" />

Sample docker-compose.yml:
version: '3'
services:
    app1:
        build: ./mobile
        ports:
            - "5001:80"
    app2:
        build: ./bank
        ports:
            - "5002:80"
    nginx:
        build: ./nginx
        ports:
            - "8080:80"
        depends_on:
            - app1
            - app2

Step 7: Run Containers
sudo docker-compose up -d
<img width="1920" height="1080" alt="Screenshot (701)" src="https://github.com/user-attachments/assets/435370da-dded-4156-92e0-5eed4650f60a" />

Check running containers:
sudo docker ps
<img width="1920" height="1080" alt="Screenshot (702)" src="https://github.com/user-attachments/assets/def51712-d846-40fc-bc47-5ab75ce34c7c" />

Step 8: Open Required Ports in AWS Security Group
<img width="1920" height="1080" alt="Screenshot (703)" src="https://github.com/user-attachments/assets/614d6d17-9d39-40dc-a4ff-9e0b9a02ba9b" />

Open ports for browser access:

8080 → NGINX gateway

5001 → Mobile app (optional)

5002 → Bank app (optional)

Go to EC2 → Security Groups → Inbound Rules → Add TCP rules

Step 9: Test Access Locally (EC2)
curl http://localhost:8080   # NGINX Load Balancer
curl http://localhost:5001   # Mobile app
curl http://localhost:5002   # Bank app
Step 10: Access from Browser

Use EC2 Public IPv4:
<img width="1920" height="1080" alt="Screenshot (704)" src="https://github.com/user-attachments/assets/c5e99dab-2a18-451c-ae25-55914773a01c" />

http://<EC2-Public-IP>:8080   # Load balancer

http://<EC2-Public-IP>:5001   # Mobile app
<img width="1920" height="1080" alt="Screenshot (705)" src="https://github.com/user-attachments/assets/76b0a446-c53e-400e-afd8-649523190b9e" />

http://<EC2-Public-IP>:5002   # Bank app
<img width="1920" height="1080" alt="Screenshot (706)" src="https://github.com/user-attachments/assets/1a54ca2b-5bed-4442-aea5-18c3d218e940" />

Step 11: Restart NGINX / Containers (if needed)
sudo docker-compose restart nginx
Or restart all:
sudo docker-compose restart

