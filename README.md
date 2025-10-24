# Complete Step-by-Step Guide: Self-Hosted GitHub Runner with Docker

This guide will help you set up a self-hosted GitHub Actions runner using Docker for any repository.

---

## 📋 Prerequisites

- Docker and Docker Compose installed
- A GitHub account with access to a repository
- Basic command line knowledge

---

## 🔑 Step 1: Create a GitHub Personal Access Token (Classic)

1. **Go to GitHub Settings:**
   - Click your profile picture (top right) → **Settings**
   - Scroll down to **Developer settings** (bottom left)
   - Click **Personal access tokens** → **Tokens (classic)**

2. **Generate New Token:**
   - Click **"Generate new token (classic)"**
   - Give it a descriptive name: `Self-hosted Runner Token`
   - Set expiration: Choose based on your needs (30/60/90 days or No expiration)

3. **Select Permissions:**
   - ✅ **`repo`** - Full control of private repositories (this includes all sub-permissions)
   - ✅ **`workflow`** - Update GitHub Action workflows

4. **Generate and Copy:**
   - Click **"Generate token"** at the bottom
   - **⚠️ IMPORTANT:** Copy the token immediately - it starts with `ghp_`
   - You won't be able to see it again!

---

## 📁 Step 2: Set Up Your Project Structure

Create a directory for your runner configuration:

```bash
mkdir github-runner-setup
cd github-runner-setup
```

---

## 🐳 Step 3: Create Docker Compose File

Create a file named `docker-compose.yml`:

```bash
nano docker-compose.yml
```

Paste this content:

```yaml
version: "3.8"

services:
  gh-runner:
    image: myoung34/github-runner:latest
    restart: unless-stopped
    environment:
      REPO_URL: https://github.com/<owner>/<repo>  # CHANGE THIS
      RUNNER_NAME_PREFIX: docker-runner-
      RUNNER_LABELS: self-hosted,linux,x64,docker
      RUNNER_GROUP: default
      ACCESS_TOKEN: ${ACCESS_TOKEN}
      EPHEMERAL: "true"
      DISABLE_AUTO_UPDATE: "true"
      RUNNER_WORKDIR: /_work
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - ghrunner-data:/_work

volumes:
  ghrunner-data:
```

**Update the `REPO_URL`** with your actual repository:

- Example: `https://github.com/sudo455/github-runner-tester`

---

## 🔐 Step 4: Create Environment File

Create a `.env` file in the same directory:

```bash
nano .env
```

Add your token (replace with your actual token):

```bash
ACCESS_TOKEN=ghp_your_actual_token_here_xxxxxxxxxxxxxxxxxxxx
```

**Security tip:** Add `.env` to `.gitignore` if you're version controlling this:

```bash
echo ".env" >> .gitignore
```

---

## 🚀 Step 5: Start the Runner

Start the runner container:

```bash
docker compose up -d
```

Check if it's running:

```bash
docker compose ps
```

View logs to confirm registration:

```bash
docker compose logs -f gh-runner
```

You should see:

```
✓ Connected to GitHub
✓ Runner successfully added
✓ Settings Saved
Listening for Jobs
```

Press `Ctrl+C` to exit logs.

---

## ✅ Step 6: Verify Runner on GitHub

1. Go to your repository on GitHub
2. Navigate to: **Settings** → **Actions** → **Runners**
3. You should see your runner listed with a name like `docker-runner-xxxxx`
4. Status should show a **green dot** (Idle)

---

## 📝 Step 7: Create a Workflow File

In your repository, create the workflow directory structure:

```bash
mkdir -p .github/workflows
```

Create a workflow file:

```bash
nano .github/workflows/self-hosted-demo.yml
```

Paste this content:

```yaml
name: Demo on self-hosted Docker runner

on:
  push:
    branches: [ "main" ]
  workflow_dispatch:  # Allows manual triggering

jobs:
  hello:
    runs-on: [self-hosted]
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Display system info
        run: echo "Hello from $(uname -a)"
      
      - name: Test Docker access
        run: |
          echo "Testing Docker functionality..."
          docker version
          docker run --rm alpine sh -c "echo '✅ Container executed successfully!' && uname -a"
      
      - name: Verify workspace
        run: |
          echo "Working directory: $(pwd)"
          ls -la
```

---

## 🧪 Step 8: Test Your Workflow

**Option A: Push to trigger**

```bash
git add .github/workflows/self-hosted-demo.yml
git commit -m "Add self-hosted runner workflow"
git push origin main
```

**Option B: Manual trigger**

1. Go to your repo on GitHub
2. Click **Actions** tab
3. Select your workflow
4. Click **"Run workflow"** → **"Run workflow"**

---

## 📊 Step 9: Monitor Execution

1. Go to the **Actions** tab in your repository
2. Click on the running workflow
3. Watch it execute on your self-hosted runner!

You can also watch live logs from your server:

```bash
docker compose logs -f gh-runner
```

---

## 🛠️ Common Management Commands

### View runner logs

```bash
docker compose logs -f gh-runner
```

### Restart runner

```bash
docker compose restart gh-runner
```

### Stop runner

```bash
docker compose down
```

### Start runner

```bash
docker compose up -d
```

### Update runner image

```bash
docker compose pull
docker compose up -d
```

### Remove everything (including data)

```bash
docker compose down -v
```

---

## 🔧 Troubleshooting

### Runner not appearing on GitHub?

- Check logs: `docker compose logs gh-runner`
- Verify token permissions (needs `repo` scope)
- Ensure `REPO_URL` is correct

### Authentication errors?

- Regenerate token with correct permissions
- Update `.env` file with new token
- Restart: `docker compose restart gh-runner`

### Workflow not running?

- Check Actions are enabled: Settings → Actions → General
- Verify runner is "Idle" (green dot) in Settings → Actions → Runners
- Make sure workflow file is on the `main` branch

### Docker permission errors in jobs?

- The runner has access via `/var/run/docker.sock`
- On SELinux systems (Fedora/RHEL), add `:Z` to volumes

---

## 🎯 Next Steps

Now you can:

- **Build and test Docker images** in your workflows
- **Run integration tests** with multiple containers
- **Deploy applications** from your CI/CD pipeline
- **Use any Docker-based tools** in your workflows

### Example: Building a Docker image in your workflow

```yaml
- name: Build Docker image
  run: |
    docker build -t myapp:latest .
    docker images
```

---

## 📚 Additional Configuration Options

### Multiple runners

Scale your runner service:

```bash
docker compose up -d --scale gh-runner=3
```

### Organization-level runners

Change `REPO_URL` to:

```yaml
REPO_URL: https://github.com/your-org
```

(Requires org admin permissions and `admin:org` token scope)

### Custom labels

Modify in docker-compose.yml:

```yaml
RUNNER_LABELS: self-hosted,linux,x64,docker,gpu,production
```

Then use in workflow:

```yaml
runs-on: [self-hosted, gpu, production]
```

---

## 🔒 Security Best Practices

1. **Rotate tokens regularly** (set expiration dates)
2. **Never commit `.env` file** to git
3. **Use ephemeral runners** (`EPHEMERAL: "true"`) - automatically deregisters after each job
4. **Keep Docker images updated** regularly
5. **Monitor runner activity** in GitHub Settings → Actions → Runners
6. **Limit runner to specific workflows** if needed (using labels)

---

That's it! You now have a fully functional self-hosted GitHub Actions runner running in Docker. 🎉
