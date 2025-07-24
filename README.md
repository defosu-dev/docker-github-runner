# GitHub Actions Self-Hosted Runner with Docker-in-Docker

A production-ready setup for running GitHub Actions self-hosted runners in Docker containers with Docker-in-Docker (DinD) support for building Docker images.

## 🚀 Features

- ✅ **Organization-level runner** - Available to all repositories in your organization
- ✅ **Docker-in-Docker support** - Build and run Docker containers in workflows  
- ✅ **Memory limits** - Configurable resource constraints
- ✅ **Security focused** - Prevents fork abuse for open-source projects
- ✅ **Ephemeral mode** - Auto-cleanup after each job
- ✅ **Auto-restart** - Survives server reboots
- ✅ **Easy token management** - Clear documentation for token rotation

## 📋 Requirements

- Docker & Docker Compose
- GitHub organization with admin access
- Server with at least 4GB RAM (8GB+ recommended)

## 🛠️ Quick Setup

### 1. Clone and Configure

```bash
git clone <your-repo>
cd runner
cp .env.example .env
```

### 2. Get GitHub Runner Token

1. Go to your organization settings: `https://github.com/YourOrg/settings/actions/runners`
2. Click **"New runner"**
3. Select **Linux** and **x64**
4. Copy the token from the `--token` parameter

### 3. Update Configuration

Edit `.env` file:

```bash
GITHUB_ORG=YourOrganizationName
RUNNER_TOKEN=BSYADL_your_token_here
RUNNER_NAME=your-runner-name
RUNNER_LABELS=linux,docker,self-hosted,trusted,your-org-only
```

### 4. Start the Runner

```bash
# Start containers
docker compose up -d

# Check status
docker compose logs runner -f
```

You should see: `✅ Listening for Jobs...`

## 🔧 Configuration

### Environment Variables

| Variable | Description | Default | Required |
|----------|-------------|---------|----------|
| `GITHUB_ORG` | Your GitHub organization name | - | ✅ |
| `RUNNER_TOKEN` | Registration token from GitHub | - | ✅ |
| `RUNNER_NAME` | Custom runner name | `ephemeral-org-runner` | ❌ |
| `RUNNER_LABELS` | Comma-separated labels | `linux,docker,self-hosted` | ❌ |
| `RUNNER_EPHEMERAL` | Auto-cleanup after jobs | `true` | ❌ |
| `DIND_MEMORY_LIMIT` | Docker-in-Docker memory limit | `6g` | ❌ |
| `DOCKER_STORAGE_SIZE` | Docker storage tmpfs size | `6G` | ❌ |
| `RUNNER_MEMORY_LIMIT` | Runner process memory limit | `2g` | ❌ |

### Memory Configuration

Adjust based on your server capacity:

```bash
# For 8GB+ servers
DIND_MEMORY_LIMIT=8g
DOCKER_STORAGE_SIZE=8G

# For 4GB servers  
DIND_MEMORY_LIMIT=4g
DOCKER_STORAGE_SIZE=4G
```

## 🔒 Security for Open-Source Projects

If your repositories are public, follow these security steps:

### 1. GitHub Organization Settings

Go to `https://github.com/YourOrg/settings/actions`:

- ✅ **Actions permissions**: "Allow all actions and reusable workflows"
- ❌ **Allow public repositories**: **UNCHECK** this option
- ✅ **Repository access**: Set to "Selected repositories"

### 2. Runner Access Control

1. Go to `https://github.com/YourOrg/settings/actions/runners`
2. Click on your runner name
3. Under **"Repository access"**, select **"Selected repositories"**
4. Add only your organization's repositories (not forks)

### 3. Workflow Security

Add repository owner checks to your workflows:

```yaml
jobs:
  build:
    runs-on: [self-hosted, trusted, your-org-only]
    # Only run for organization repositories
    if: github.repository_owner == 'YourOrg'
    steps:
      - uses: actions/checkout@v4
      # ... rest of your workflow
```

## 🔄 Token Management

### Continuous Running (Recommended)

Start the runner once and leave it running:

```bash
# Start once
docker compose up -d

# No token updates needed!
# Runner stays connected indefinitely
```

### Stop/Start Workflow

If you need to stop and restart after hours:

1. **Get fresh token** from GitHub (tokens expire after ~1 hour)
2. **Update `.env`** with new token
3. **Restart runner**:
   ```bash
   docker compose down && docker compose up -d
   ```

### Token Health Check

```bash
# Check for authentication issues
docker compose logs runner --tail 10 | grep -E "(401|404|Invalid|Authentication)"
```

## 📊 Monitoring

### Check Runner Status

```bash
# Container status
docker compose ps

# Resource usage
docker stats --no-stream

# Runner logs
docker compose logs runner -f

# DIND logs
docker compose logs dind -f
```

### GitHub Status

Check runner status at: `https://github.com/YourOrg/settings/actions/runners`

Should show: **"Idle"** (green) when not running jobs

## 🎯 Using the Runner

### In Your Workflows

```yaml
name: Build with Self-Hosted Runner

on:
  push:
    branches: [main, dev]

jobs:
  build:
    # Use your self-hosted runner
    runs-on: [self-hosted, trusted, your-org-only]
    
    steps:
      - name: Checkout Code
        uses: actions/checkout@v4
        
      - name: Build Docker Image
        run: |
          docker build -t myapp:${{ github.sha }} .
          
      - name: Run Tests
        run: |
          docker run --rm myapp:${{ github.sha }} npm test
```

### Available Labels

Use these labels to target your runner:

- `self-hosted` - Basic self-hosted runner
- `linux` - Linux operating system
- `docker` - Docker support available
- `trusted` - Security label for organization access
- `your-org-only` - Custom label to prevent fork usage

## 🧹 Maintenance

### Regular Cleanup

```bash
# Clean up Docker resources
docker system prune -f

# Clean build cache
docker builder prune -f

# Check disk usage
docker system df -v
```

### Update Runner Image

```bash
docker compose pull
docker compose up -d
```

### Logs Management

Logs are automatically rotated with these limits:
- **Runner**: 50MB max, 3 files
- **DIND**: 100MB max, 3 files

## 🚨 Troubleshooting

### Runner Not Connecting

1. **Check token validity**:
   ```bash
   docker compose logs runner
   ```

2. **Get fresh token** if you see 401/404 errors

3. **Verify organization settings** allow self-hosted runners

### High Memory Usage

1. **Monitor resource usage**:
   ```bash
   docker stats
   ```

2. **Adjust memory limits** in `.env` file

3. **Clean up old images**:
   ```bash
   docker image prune -f
   ```

### Workflows Stay Queued

1. **Check runner status** in GitHub organization settings
2. **Verify workflow labels** match runner labels
3. **Ensure repository has access** to organization runner

## 📚 Useful Commands

```bash
# Start runner
docker compose up -d

# Stop runner  
docker compose down

# View logs
docker compose logs runner -f

# Check status
docker compose ps

# Restart with fresh config
docker compose down && docker compose up -d

# Monitor resources
watch -n 5 'docker stats --no-stream'

# Health check
docker compose logs runner --tail 10 | grep -E "(Listening|ERROR|401|404)"
```

## 🤝 Contributing

1. Fork the repository
2. Create your feature branch
3. Test your changes
4. Submit a pull request

## 📄 License

This project is licensed under the MIT License - see the LICENSE file for details.

## 🔗 References

- [GitHub Self-Hosted Runners Documentation](https://docs.github.com/en/actions/hosting-your-own-runners)
- [Docker-in-Docker Documentation](https://hub.docker.com/_/docker)
- [Actions Runner Docker Image](https://github.com/summerwind/actions-runner)
