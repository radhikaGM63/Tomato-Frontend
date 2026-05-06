# 🍅 Tomato - Food Delivery Frontend

A modern food delivery web application built with **React.js + Vite**, 
containerized with **Docker**, and deployed automatically using 
**Jenkins CI/CD Pipeline** on **AWS EC2**.

## 🌐 Live Application
http://<Public_IP>

## 📖 Project Overview

Tomato is a food delivery frontend application where users can:
- Browse food items by category
- Add/remove items from cart
- Place food orders

But more importantly, this project demonstrates a **complete real-world DevOps workflow**:
VS Code → GitHub → Jenkins CI/CD → Docker → AWS EC2 → Live App


## 🛠️ Tech Stack

### Frontend
| Technology | Purpose |
|------------|---------|
| React.js | Frontend framework |
| Vite | Build tool |
| React Context API | Global state (cart management) |
| React Router DOM | Page routing |
| CSS3 | Styling |

### DevOps
| Technology | Purpose |
|------------|---------|
| Git & GitHub | Source code management |
| Docker | Containerization |
| Nginx | Serve production build |
| Jenkins | CI/CD automation |
| AWS EC2 | Cloud hosting |
| Amazon Linux 2023 | EC2 operating system |
| GitHub Webhooks | Auto-trigger Jenkins pipeline |


## 📁 Project Structure

food-del/
│
├── public/                     # Static assets
│   ├── header_img.png
│   └── vite.svg
│
├── src/                        # Main source code
│   ├── assets/                 # Images
│   ├── components/             # Reusable components
│   │   ├── Navbar/
│   │   ├── Footer/
│   │   ├── FoodItem/
│   │   ├── FoodDisplay/
│   │   ├── ExploreMenu/
│   │   └── LoginPopup/
│   ├── context/
│   │   └── StoreContext.jsx    # Global state management
│   ├── pages/
│   │   ├── Home/
│   │   ├── Cart/
│   │   └── PlaceOrder/
│   ├── App.jsx
│   ├── main.jsx
│   └── index.css
│
├── Dockerfile                  # Docker configuration ✅
├── Jenkinsfile                 # Jenkins pipeline ✅
├── index.html
├── package.json
├── vite.config.js
└── README.md

## 🚀 Complete Process — What We Did (Step by Step)

This section explains the **exact journey** of how this project was 
built and deployed from scratch.

### ✅ STEP 1 — Project Was Ready in VS Code

- The React + Vite project **Tomato Frontend** was already built 
  and running locally in VS Code
- Project had all components: Navbar, FoodDisplay, Cart, PlaceOrder etc.
- It was running locally using `npm run dev` at `http://localhost:5173`

---

### ✅ STEP 2 — Created Dockerfile

Inside the project root folder in VS Code, we created a `Dockerfile`

**Why Dockerfile?**
- To package our React app into a Docker container
- Stage 1 builds the app using Node.js
- Stage 2 serves it using lightweight Nginx
- Final image is small (~50MB) with no Node.js bloat
- Since project uses Vite, build output is `/dist` (not `/build`)

### ✅ STEP 3 — Created Jenkinsfile

Inside the project root folder in VS Code, we created a `Jenkinsfile`

**Why Jenkinsfile?**
- Defines all the CI/CD pipeline stages
- Jenkins reads this file automatically from GitHub
- Every stage runs in order — install, build, dockerize, deploy
- `|| true` prevents pipeline failure if container doesn't exist yet

### ✅ STEP 4 — Pushed Code to GitHub

After creating Dockerfile and Jenkinsfile, we pushed everything to GitHub:
**GitHub Repository now had:**
- ✅ All React source code
- ✅ Dockerfile
- ✅ Jenkinsfile
- ✅ package.json, vite.config.js etc.

### ✅ STEP 5 — Launched AWS EC2 Instance

We created a cloud server on AWS to host Jenkins and our app:

### ✅ STEP 6 — Connected to EC2 via SSH

From local terminal, we connected to EC2:

### ✅ STEP 7 — Installed All Required Packages on EC2

We installed everything Jenkins and Docker need:

# Update system
sudo yum update -y

# Install Java 17 (Jenkins needs Java)
sudo yum install -y java-17-amazon-corretto

# Install Jenkins
sudo wget -O /etc/yum.repos.d/jenkins.repo \
  https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key
sudo yum install -y jenkins
sudo systemctl start jenkins
sudo systemctl enable jenkins

# Install Git
sudo yum install -y git

# Install Docker
sudo yum install -y docker
sudo systemctl start docker
sudo systemctl enable docker

# Give Jenkins permission to use Docker
sudo usermod -aG docker jenkins
sudo usermod -aG docker ec2-user
sudo chmod 666 /var/run/docker.sock

# Install libatomic (required for Node.js on Amazon Linux)
sudo yum install -y libatomic

# Install Node.js 20
curl -fsSL https://rpm.nodesource.com/setup_20.x | sudo bash -
sudo yum install -y nodejs

# Restart Jenkins

sudo systemctl restart jenkins

### ✅ STEP 8 — Configured Jenkins

1. Opened Jenkins at `http://YOUR_EC2_PUBLIC_IP:8080`
2. Got initial password:
sudo cat /var/lib/jenkins/secrets/initialAdminPassword

3. Installed suggested plugins
4. Created admin user
5. Installed additional plugins:
   - NodeJS Plugin
   - Docker Plugin
   - Docker Pipeline Plugin
   - GitHub Integration Plugin
6. Configured NodeJS Tool:
   - Manage Jenkins → Tools → NodeJS → Add → Name: `NodeJS20` → Version: `20.x`

### ✅ STEP 9 — Created Jenkins Pipeline Job

1. New Item → `food-app` → Pipeline → OK
2. Pipeline script from SCM → Git
3. Repository URL: `https://github.com/paladugubharath/Tomato-Frontend.git`
4. Branch: `*/main`
5. Script Path: `Jenkinsfile`
6. Saved

### ✅ STEP 10 — Configured GitHub Webhook

So that every `git push` automatically triggers Jenkins:

1. GitHub repo → Settings → Webhooks → Add webhook
2. Payload URL: `http://YOUR_EC2_PUBLIC_IP:8080/github-webhook/`
3. Content type: `application/json`
4. Event: Just the push event
5. In Jenkins job → Build Triggers → ✅ GitHub hook trigger for GITScm polling

### ✅ STEP 11 — Fixed Errors During First Build

During first build we faced 2 errors and fixed them:

**Error 1: `npm: command not found`**
- Cause: NodeJS tool not added in Jenkinsfile
- Fix: Added `tools { nodejs 'NodeJS20' }` in Jenkinsfile

**Error 2: `libatomic.so.1: No such file`**
- Cause: Missing library on Amazon Linux for Node.js
- Fix: `sudo yum install -y libatomic`
- Also changed NodeJS version from 25 to stable 20.x in Jenkins tools

### ✅ STEP 12 — Build Success & App Deployed! 🎉

After fixes, Jenkins pipeline ran successfully:

✅ Stage 1: Clone Repository     — pulled code from GitHub
✅ Stage 2: Install Dependencies  — npm install
✅ Stage 3: Build React App       — npm run build (output: /dist)
✅ Stage 4: Build Docker Image    — docker build -t food-del-app .
✅ Stage 5: Stop Old Container    — docker stop/rm (cleanup)
✅ Stage 6: Run New Container     — docker run -d -p 80:80

**App went live at:**

http://<EC2_PUBLIC_IP>


## 🔄 Final CI/CD Flow

┌──────────────────────────────────────────────────┐
│             DEVELOPER (VS Code)                   │
│  Write code → git add → git commit → git push    │
└─────────────────────┬────────────────────────────┘
                      │
                      ▼
┌──────────────────────────────────────────────────┐
│                  GITHUB                           │
│  Receives push → Fires Webhook to Jenkins        │
└─────────────────────┬────────────────────────────┘
                      │
                      ▼
┌──────────────────────────────────────────────────┐
│           JENKINS (AWS EC2 :8080)                 │
│                                                  │
│  📥 Clone Repo                                   │
│  📦 npm install                                  │
│  🔨 npm run build                                │
│  🐳 docker build                                 │
│  🛑 docker stop old container                    │
│  🚀 docker run new container                     │
└─────────────────────┬────────────────────────────┘
                      │
                      ▼
┌──────────────────────────────────────────────────┐
│         DOCKER CONTAINER (Port 80)                │
│   Nginx serving React production build           │
│   🌐 http://YOUR_EC2_PUBLIC_IP                  │
└──────────────────────────────────────────────────┘

## 🐛 Errors Faced & Solutions

| # | Error | Cause | Solution |
|---|-------|-------|----------|
| 1 | `npm: command not found` | NodeJS not in Jenkinsfile tools | Added `tools { nodejs 'NodeJS20' }` |
| 2 | `libatomic.so.1 not found` | Missing lib on Amazon Linux | `sudo yum install -y libatomic` |
| 3 | `git add` not staging | Files cached in git | `git rm -r --cached . && git add .` |

---

## 💻 How to Run Locally
# Clone the repo
git clone https://github.com/paladugubharath/Tomato-Frontend.git
cd food-del

# Install dependencies
npm install

# Start dev server
npm run dev
App runs at: `http://localhost:5173`

## 🐳 How to Run with Docker
# Build image
docker build -t food-del-app .

# Run container
docker run -d -p 80:80 --name food-del-container food-del-app

# Open in browser
http://localhost

## 👨‍💻 Author

**Bharath Paladugu**
- GitHub:   https://github.com/paladugubharath
- LinkedIn: coming soon...
