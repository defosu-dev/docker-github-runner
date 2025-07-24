# GitHub Actions Self-Hosted Runner

A Docker-based setup for running GitHub Actions self-hosted runners with Docker-in-Docker support.

## 🚀 Features

- Organization-level runner for all repositories
- Docker-in-Docker support for building containers
- Configurable memory limits
- Auto-restart capability

## 📋 Requirements

- Docker & Docker Compose
- GitHub organization admin access
- 4GB+ RAM recommended

## 🛠️ Setup

### 1. Configure Environment

```bash
cp .env.example .env
```

Edit `.env` with your values:

```bash
GITHUB_ORG=YourOrganizationName
RUNNER_TOKEN=your_token_from_github
RUNNER_NAME=your-runner-name
```

### 2. Get Runner Token

1. Go to `https://github.com/YourOrg/settings/actions/runners`
2. Click "New runner" → Linux x64
3. Copy the token from `--token` parameter

### 3. Start Runner

```bash
docker compose up -d
```

Check status: `docker compose logs runner -f`

## 🔧 Configuration

| Variable | Description | Default |
|----------|-------------|---------|
| `GITHUB_ORG` | Organization name | - |
| `RUNNER_TOKEN` | Registration token | - |
| `RUNNER_NAME` | Runner name | `ephemeral-org-runner` |
| `RUNNER_LABELS` | Runner labels | `linux,docker,self-hosted` |
| `DIND_MEMORY_LIMIT` | Docker memory limit | `6g` |
| `DOCKER_STORAGE_SIZE` | Docker storage size | `6G` |
| `RUNNER_MEMORY_LIMIT` | Runner memory limit | `2g` |

## 🔄 Token Management

**Continuous Running (Recommended):**
- Start once with `docker compose up -d`  
- Leave running - no token updates needed

**After Stopping:**
- Get fresh token if stopped for hours
- Update `.env` and restart

## 🎯 Usage

Use in workflows:

```yaml
jobs:
  build:
    runs-on: self-hosted
    steps:
      - uses: actions/checkout@v4
      - run: docker build -t myapp .
```

## 📊 Commands

```bash
# Start/stop
docker compose up -d
docker compose down

# Monitor
docker compose logs runner -f
docker compose ps
docker stats

# Cleanup
docker system prune -f
```

## 🚨 Troubleshooting

**Runner not connecting:** Check token validity with `docker compose logs runner`

**High memory usage:** Adjust limits in `.env` file

**Workflows queued:** Verify runner shows "Idle" in GitHub settings

## 📚 References

- [GitHub Self-Hosted Runners](https://docs.github.com/en/actions/hosting-your-own-runners)
- [Docker-in-Docker](https://hub.docker.com/_/docker)
