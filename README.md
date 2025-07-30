# CI-CD-Pipeline
# 1. CI/CD 
**💡 Goal:** Faster, automated, reliable software delivery with minimum manual work.
## CI (Continuous Integration):
- Developers push code → code is automatically tested, built, and integrated with main branch.
- CI → build + test automation.

## CD (Continuous Delivery/Deployment):
- After CI passes → code is automatically deployed to a server (staging/production).
- CD → deploy automation.

# 2. Workflow
**Workflow →** *YAM*L file that defines CI/CD logic in GitHub Actions.
## 1. Developer 
Code Commit → Git Repo (GitHub, GitLab, Bitbucket)
## 2. CI Pipeline:
- Checkout code
- Install dependencies
- Linting & Testing
- Build artifact (e.g., Docker image)
- Store in registry
## 3. CD Pipeline:
- Deploy to staging
- Run integration tests
- (Optional) Approval step
- Deploy to production
## 4. Monitor & Rollback 
if failure.

# 3. Example Flow
**Developer → Git Push → CI (Test + Build + Docker Image) → Push to Registry → CD (Deploy to Staging → Prod) → Monitoring → Rollback if needed.**

# 4. Key Concepts
```
# 1. Workflow
Automated process defined in .github/workflows/ folder.
Written in YAML.

# 2. Job
A set of steps that run on the same machine.
Example: build → test → deploy.

# 3.Step
An individual task inside a job.
Can run commands (like npm install) or use prebuilt Actions.

# 4. Runner
The server where jobs run.
GitHub provides free runners (Linux, Windows, macOS). You can also self-host.

# 5.Action
A reusable piece of code that performs a task (e.g., checkout code, setup Node, deploy).
```
# 5. Tools
- **Version Control:-** GitHub, GitLab, Bitbucket
- **CI/CD:-** GitHub Actions, Jenkins, GitLab CI, CircleCI
- **Artifacts Repositories:-** DockerHub, Nexus, AWS ECR, JFrog Artifactory
- **Infra/Deploy:-** Docker, Kubernetes, Terraform, Ansible, AWS/GCP/Azure
- **Monitoring:-** Prometheus, Grafana, ELK, Datadog
---
---

# 1. CI/CD with GitHub Actions
- GitHub Actions is an automation platform built into GitHub.
- Whenever something happens in my repo, do this set of tasks automatically.

## 1. Workflow in GitHub Actions
- **Definition:--** GitHub’s built-in CI/CD service.
- **Workflow--** = `YAML` file (`.github/workflows/ci-cd.yml`).
- **Runs on Events:--** push, pull_request, schedule, etc.

## 🟢 Example 1: CI (Test on Every Push) 
`.github/workflows/ci.yml`
```
name: CI Pipeline

on: 
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
      
      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.10'
      
      - name: Install dependencies
        run: pip install -r requirements.txt
      
      - name: Run tests
        run: pytest
```
 🔎 What happens here:
- Runs whenever someone pushes or creates PR to main.
- Checks out repo → installs dependencies → runs tests.
   
## 🟢 Example 2: CD (Deploy to AWS EC2 After CI) 
`.github/workflows/cd.yml`
```
name: Deploy to AWS

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Deploy to EC2
        run: |
          ssh -i ${{ secrets.AWS_KEY }} ec2-user@your-server-ip "
          cd /var/www/myapp &&
          git pull origin main &&
          systemctl restart myapp"
```
🔎 Here:
- On push to main, it SSHs into your server.
- Pulls latest code.
- Restarts app service.
  
## 🟢 Example 3: CI/CD Workflow for Python (Combined)
📌 Save this file as:
- `.github/workflows/python-ci-cd.yml`
```
# ---------------------------
# GitHub Actions Workflow for Python Project
# Performs CI (lint, test) and CD (deploy to server)
# ---------------------------

name: Python CI/CD Pipeline

on:
  push:
    branches: [ main, dev ]      # Run pipeline when pushing to main or dev
  pull_request:
    branches: [ main ]           # Run checks when PR is opened against main

jobs:
  # ---------------------------
  # 1. Continuous Integration (CI) Job
  # ---------------------------
  ci:
    runs-on: ubuntu-latest       # Use latest Ubuntu runner
    strategy:
      matrix:
        python-version: [ "3.9", "3.10", "3.11" ]   # Test against multiple Python versions

    steps:
      # Step 1: Get code from repo
      - name: Checkout repository
        uses: actions/checkout@v4

      # Step 2: Set up Python environment
      - name: Set up Python ${{ matrix.python-version }}
        uses: actions/setup-python@v5
        with:
          python-version: ${{ matrix.python-version }}

      # Step 3: Install dependencies
      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip      # Upgrade pip
          pip install -r requirements.txt          # Install project dependencies
          pip install pytest flake8 black          # Install testing and linting tools

      # Step 4: Run linting check with flake8
      - name: Run Linter (flake8)
        run: flake8 .

      # Step 5: Check code formatting with black
      - name: Check code formatting (black)
        run: black --check .

      # Step 6: Run unit tests with pytest
      - name: Run Tests
        run: pytest --maxfail=1 --disable-warnings -q

  # ---------------------------
  # 2. Continuous Deployment (CD) Job
  # ---------------------------
  cd:
    runs-on: ubuntu-latest
    needs: ci                               # Run only if CI job succeeds
    if: github.ref == 'refs/heads/main'     # Deploy only when pushing to main branch

    steps:
      # Step 1: Checkout repo again (not strictly needed, but good practice)
      - name: Checkout repository
        uses: actions/checkout@v4

      # Step 2: Deploy to remote server
      - name: Deploy to Server
        run: |
          echo "🚀 Starting Deployment..."
          ssh -o StrictHostKeyChecking=no -i ${{ secrets.SSH_KEY }} \
             ${{ secrets.SERVER_USER }}@${{ secrets.SERVER_IP }} "
             cd /var/www/myapp &&                          # Go to app folder
             git pull origin main &&                       # Pull latest code
             source venv/bin/activate &&                   # Activate virtualenv
             pip install -r requirements.txt &&            # Update dependencies
             systemctl restart myapp                       # Restart app service
          "

```
🔎 Here:
- Job 1: build-test → runs tests.
- Job 2: deploy → runs only if build-test passes.
- Deploys to Vercel automatically.

## 3. 🚀 Benefits of GitHub Actions
- Integrated with GitHub (no extra setup).
- Supports Linux, Windows, macOS.
- Free usage (2,000–3,000 minutes/month).
- Marketplace with 10,000+ ready-made actions.

---
---
# 2. CI/CD with Jenkins
Jenkins is a self-hosted automation server. You install Jenkins on your server/VM.

## Pipeline Steps
- **Definition:--** Open-source automation server for CI/CD.
- **Jenkins Pipeline:--** Scripted workflow (Jenkinsfile).
- **Plugins:--** Integrate with GitHub, Docker, Kubernetes, etc.

👉 Basic (Jenkinsfile) Example

```
pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/user/repo.git'
            }
        }

        stage('Build') {
            steps {
                sh 'pip install -r requirements.txt'
            }
        }

        stage('Test') {
            steps {
                sh 'pytest'
            }
        }

        stage('Docker Build & Push') {
            steps {
                sh 'docker build -t myapp:latest .'
                sh 'docker push mydockerhub/myapp:latest'
            }
        }

        stage('Deploy') {
            steps {
                sshagent(['server-ssh']) {
                    sh 'ssh user@server "docker pull mydockerhub/myapp:latest && docker run -d -p 8000:8000 myapp:latest"'
                }
            }
        }
    }
}
```
✅ More customizable, best for enterprise-scale projects.

# Key Differences b/w GitHub Actions and Jenkins
| Feature       | GitHub Actions 🚀         | Jenkins ⚙️                                          |
| ------------- | ------------------------- | --------------------------------------------------- |
| Hosting       | Cloud (GitHub-hosted)     | Self-hosted (your server/VM)                        |
| Setup         | Easy, YAML-based          | Complex, needs Jenkins setup/plugins                |
| Cost          | Free (limited minutes)    | Free, but needs infra                               |
| Scalability   | Limited to GitHub runners | Very scalable (distributed agents)                  |
| Integration   | Native with GitHub        | Works with any VCS (GitHub, GitLab, Bitbucket, SVN) |
| Best Use Case | Small/medium projects     | Large/enterprise projects                           |

# CI/CD Workflow (General)
- **Code Commit →** Developer pushes code to GitHub (or Git).
- **Build →** CI server (Jenkins/GitHub Actions) compiles and builds.
- **Test →** Run automated tests (unit, integration, security).
- **Package →** Create artifacts (Docker images, .jar/.war, etc.).
- **Deploy →** Push to staging/prod server, Kubernetes, or cloud.
- **Monitor →** Logs, metrics, alerting.





