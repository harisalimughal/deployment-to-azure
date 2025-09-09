[![Review Assignment Due Date](https://classroom.github.com/assets/deadline-readme-button-22041afd0340ce965d47ae6ef1cefeee28c7c493a6346c4f15d667ab976d596c.svg)](https://classroom.github.com/a/4JMUHqcs)
# Lecture No 3


**Tutorial: Deployment to Azure with GitHub Actions**

### Introduction

In the last lecture, you learned about **branching** and setting up a **CI pipeline** in GitHub Actions.

Now we’ll take it a step further: once your code passes tests, it should also be **deployed automatically** to a cloud platform.

We’ll use **Azure Web App for Linux** to host our Flask app. GitHub Actions will be configured to deploy the app whenever changes are merged into `main`.

---

## Example Scenario

* You have a Flask web application in GitHub.
* The app runs locally and passes all tests via CI.
* Now, when you merge changes into `main`, GitHub Actions should deploy the app to Azure.

---

## Steps

### 1. Create an Azure Web App

1. Go to [Azure Portal](https://portal.azure.com/).
2. Create a new **App Service → Web App**.

   * Runtime stack: **Python 3.10**
   * Region: choose nearest region.
   * Operating System: Linux.
3. Note the **App Name** (e.g., `mlops-calc-fall2025`).

---

### 2. Configure Deployment Credentials

1. In Azure Portal → Web App → **Deployment Center**.
2. Select **GitHub Actions** as the CI/CD option.
3. Authorize your GitHub account and select the repository.
4. Azure will auto-generate a GitHub Actions workflow file.

---

### 3. Add Azure Publish Profile to GitHub Secrets

1. In Web App → **Deployment Center → Settings → Publish Profile**.
2. Download the publish profile (`.PublishSettings` file).
3. Go to GitHub → Repo → **Settings → Secrets → Actions**.
4. Add a new secret:

   * Name: `AZURE_WEBAPP_PUBLISH_PROFILE`
   * Value: copy the contents of the downloaded file.

---

### 4. Create Deployment Workflow

Inside your repo, create: **`.github/workflows/deploy.yml`**

```yaml
name: Deploy to Azure Web App

on:
  push:
    branches:
      - main

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: "3.10"

      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install -r requirements.txt

      - name: Run tests
        run: pytest

      - name: Deploy to Azure WebApp
        uses: azure/webapps-deploy@v2
        with:
          app-name: flask-demo-12345   # Replace with your Azure Web App name
          publish-profile: ${{ secrets.AZURE_WEBAPP_PUBLISH_PROFILE }}
          package: .
```

---

## Verification

1. Push changes to `main`.
2. GitHub Actions will run tests.
3. If successful, it will deploy the app to Azure.
4. Open `https://mlops-calc-fall2025.azurewebsites.net/` to confirm deployment.

---

## Exercises

### **Exercise 1: First Deployment**

* Set up an Azure Web App (Python 3.10).
* Connect your GitHub repo with Azure.
* Deploy the Flask app and verify it runs in the browser.

---

### **Exercise 2: Feature Branch Deployment**

* Create a branch `feature/hello-route`.
* Add a new `/hello` route in Flask returning `"Hello from Azure!"`.
* Push and merge into `main`.
* Verify the deployed app on Azure reflects the change.

---

### **Exercise 3: Deployment Fails**

* Intentionally break the Flask app (syntax error).
* Push changes → see GitHub Actions fail **before deployment**.
* Fix the bug and redeploy successfully.

---

### **Exercise 4: Extend Pipeline with Linting**

* Add a linting step using `flake8` in `deploy.yml`.
* Push code with style issues → deployment should be blocked.
* Fix and confirm clean deployment.

---

## Key Takeaways

* GitHub Actions can handle **CI (testing)** and **CD (deployment)** together.
* Azure Web Apps make Flask deployment simple using publish profiles.
* Failing tests or linting errors **stop deployment automatically**.
* Continuous Deployment ensures your `main` branch is always live on the cloud.
