this is for cicd
# My Project
This is a flood detection project using YOLOv8.

⚙️ GITHUB ACTIONS — CI/CD STEP-BY-STEP
1️⃣ Setup — Create Workflow File

Create the directory and YAML file in your repo:

.github/workflows/ci-cd.yml


This YAML file defines all your automation steps (CI/CD pipeline).

2️⃣ Define the Trigger (When to Run)

Tell GitHub when to start the workflow.

on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main


🔹 Runs when you push or create a PR to the main branch.
(You can also trigger manually, on schedule, etc.)

3️⃣ Define Jobs

Each job runs in a fresh virtual machine (like Ubuntu, Windows, etc.)

jobs:
  build:
    runs-on: ubuntu-latest

4️⃣ Step 1: Checkout Code

GitHub needs to clone your repo in the workflow environment.

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

5️⃣ Step 2: Set Up Environment

Install the programming language and dependencies.

Example for Python:

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.10'

6️⃣ Step 3: Install Dependencies

Use your requirements.txt or environment manager.

      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install -r requirements.txt

7️⃣ Step 4: Code Quality / Linting

Ensure your code follows formatting standards.

      - name: Run Black formatter
        run: black --check .


Or run Flake8, Pylint, etc.

8️⃣ Step 5: Run Unit Tests (CI Validation)

Execute your tests to ensure the code works.

      - name: Run tests
        run: pytest tests/


If tests fail → pipeline fails 🚫
If pass → continues ✅

9️⃣ Step 6: Build / Train / Package

Build the project, or in ML, train & store models.

      - name: Build or Train
        run: python train_model.py

🔟 Step 7: Upload Artifacts

Save the build outputs or model weights.

      - name: Upload model artifact
        uses: actions/upload-artifact@v4
        with:
          name: model
          path: model.pkl

1️⃣1️⃣ Step 8: Continuous Deployment (CD)

Deploy to a target (like AWS, GCP, Azure, DockerHub, etc.)

Example: Deploy to DockerHub

      - name: Log in to DockerHub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}

      - name: Build and Push Docker image
        run: |
          docker build -t my-app:latest .
          docker tag my-app:latest myuser/my-app:latest
          docker push myuser/my-app:latest

1️⃣2️⃣ Step 9: Deploy to Production or Cloud

You can deploy to:

GitHub Pages

AWS S3 / Lambda

Azure App Service

Google Cloud Run / Vertex AI

Kubernetes / Docker Compose

Example (for AWS EC2):

      - name: Deploy to EC2
        run: |
          scp -i key.pem app.py ubuntu@your-ip:/home/ubuntu/
          ssh -i key.pem ubuntu@your-ip 'sudo systemctl restart app'

1️⃣3️⃣ Step 10: Notifications (Optional)

Send Slack, Discord, or Email alerts.

      - name: Notify Slack
        uses: slackapi/slack-github-action@v2
        with:
          payload: '{"text":"✅ Deployment successful!"}'

---------------------------------------------------------------------------------------------
---------------------------------------------------------------------------------------------


name: CI/CD Pipeline - Flood Detection YOLOv8

on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main

jobs:
  ci-cd:
    runs-on: ubuntu-latest

    steps:
      # 1️⃣ Checkout Repository
      - name: Checkout repository
        uses: actions/checkout@v4

      # 2️⃣ Setup Python Environment
      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.10'

      # 3️⃣ Install Dependencies
      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install -r requirements.txt

      # 4️⃣ Code Formatting / Linting (Black)
      - name: Check code formatting
        run: |
          pip install black
          black --check .

      # 5️⃣ Run Tests (CI)
      - name: Run tests
        run: |
          pip install pytest
          pytest tests/ --maxfail=1 --disable-warnings -q

      # 6️⃣ Build or Train Model
      - name: Train or Build Model
        run: |
          echo "🚀 Training YOLOv8 Flood Detection model..."
          python train_model.py

      # 7️⃣ Save Model Artifact
      - name: Upload model artifact
        uses: actions/upload-artifact@v4
        with:
          name: flood-detection-model
          path: model.pt

      # 8️⃣ Build Docker Image
      - name: Build Docker image
        run: |
          docker build -t flood-detection:latest .

      # 9️⃣ Login to DockerHub
      - name: Log in to DockerHub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}

      # 🔟 Push Docker Image
      - name: Push Docker image to DockerHub
        run: |
          docker tag flood-detection:latest ${{ secrets.DOCKER_USERNAME }}/flood-detection:latest
          docker push ${{ secrets.DOCKER_USERNAME }}/flood-detection:latest

      # 1️⃣1️⃣ Deploy to EC2 (Production)
      - name: Deploy to EC2
        if: github.ref == 'refs/heads/main'
        env:
          EC2_IP: ${{ secrets.EC2_IP }}
          EC2_USER: ${{ secrets.EC2_USER }}
          EC2_KEY: ${{ secrets.EC2_KEY }}
        run: |
          echo "$EC2_KEY" > key.pem
          chmod 600 key.pem
          scp -i key.pem -o StrictHostKeyChecking=no docker-compose.yml $EC2_USER@$EC2_IP:/home/$EC2_USER/
          ssh -i key.pem -o StrictHostKeyChecking=no $EC2_USER@$EC2_IP '
            docker pull ${{ secrets.DOCKER_USERNAME }}/flood-detection:latest &&
            docker compose up -d
          '

      # 1️⃣2️⃣ Send Notification (Optional)
      - name: Send Slack Notification
        if: always()
        uses: slackapi/slack-github-action@v2
        with:
          payload: |
            {
              "text": ":rocket: CI/CD pipeline completed for *Flood Detection YOLOv8!*",
              "attachments": [
                {
                  "color": "#36a64f",
                  "text": "Status: ${{ job.status }}\nBranch: ${{ github.ref }}\nCommit: ${{ github.sha }}"
                }
              ]
            }
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
