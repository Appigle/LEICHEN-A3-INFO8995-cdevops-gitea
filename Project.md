# Project Analysis: cdevops-gitea

## Overview

This project is a Kubernetes laboratory assignment (Assignment 3) for the INFO8995 Container and Orchestration course at Conestoga College. The project demonstrates the deployment and management of Gitea (a self-hosted Git service) on Kubernetes, with a focus on transitioning from development to production environments.

## Project Purpose

The primary objective is to deploy Gitea in a Kubernetes environment and transform it from a development setup (SQLite-based) to a production-ready deployment (MySQL-based) with persistent storage and external database connectivity.

## Directory Structure

```
LEICHEN-A3-INFO8995-cdevops-gitea/
├── .devcontainer/
│   └── devcontainer.json          # VS Code development container configuration
├── .git/                          # Git version control directory
├── gitea/                         # Gitea-specific configurations
│   ├── down.yml                   # Ansible playbook to remove Gitea deployment
│   ├── up.yml                     # Ansible playbook to deploy Gitea
│   └── values.yaml                # Helm values for Gitea configuration
├── k8s/                           # Kubernetes infrastructure (git submodule)
├── .gitignore                     # Git ignore patterns (Python-focused)
├── .gitmodules                    # Git submodule configuration
├── down.yml                       # Main orchestration playbook (teardown)
├── LICENSE                        # MIT License
├── README.md                      # Project documentation and instructions
└── up.yml                         # Main orchestration playbook (setup)
```

## File Analysis

### Core Configuration Files

#### 1. Main Orchestration (`up.yml`, `down.yml`)

**up.yml:**

```yaml
---
- import_playbook: k8s/up.yml
- import_playbook: gitea/up.yml
```

**down.yml:**

```yaml
---
- import_playbook: gitea/down.yml
- import_playbook: k8s/down.yml
```

These files serve as the main entry points for deploying and tearing down the entire infrastructure. They orchestrate the execution of Kubernetes setup followed by Gitea deployment (up) or reverse order for teardown (down).

#### 2. Gitea Configuration (`gitea/`)

**gitea/up.yml:**

- Adds the official Gitea Helm repository
- Deploys Gitea using Helm chart with custom values
- Uses Ansible's `kubernetes.core.helm` module

**gitea/down.yml:**

- Removes the Gitea Helm release
- Ensures clean teardown with wait conditions

**gitea/values.yaml:**
Current configuration (Development Mode):

```yaml
redis-cluster:
  enabled: false
redis:
  enabled: false
postgresql:
  enabled: false
postgresql-ha:
  enabled: false

persistence:
  enabled: false

gitea:
  config:
    database:
      DB_TYPE: sqlite3
    session:
      PROVIDER: memory
    cache:
      ADAPTER: memory
    queue:
      TYPE: level
```

This configuration sets up Gitea in development mode with:

- SQLite3 database (non-persistent)
- Memory-based sessions and caching
- No external dependencies (Redis, PostgreSQL)
- No persistent storage

### Hidden Files Analysis

#### 1. `.gitignore`

- Comprehensive Python-focused ignore patterns
- Covers build artifacts, virtual environments, IDE files
- Includes patterns for various Python tools (Poetry, PDM, Ruff)
- Notable exclusions: `nohup.out`, `.pypirc`

#### 2. `.gitmodules`

```ini
[submodule "k8s"]
    path = k8s
    url = https://github.com/rhildred/ansible-k8s
```

- References external Kubernetes infrastructure as a git submodule
- Points to a reusable Ansible-based Kubernetes setup repository

#### 3. `.devcontainer/devcontainer.json`

Development container configuration with:

- Docker support and security options
- K3D environment variable for lightweight Kubernetes
- VS Code extensions for:
  - Docker development
  - Python development and debugging
  - SQLite viewing
  - Code quality tools

## Technology Stack

### Core Technologies

- **Kubernetes**: Container orchestration platform
- **Ansible**: Infrastructure automation and configuration management
- **Helm**: Kubernetes package manager
- **Gitea**: Self-hosted Git service
- **Docker**: Containerization (via dev containers)

### Development vs Production Stack

**Development (Current):**

- SQLite3 database
- Memory-based sessions/caching
- No persistence
- Lightweight, local development focus

**Production (Target):**

- External MySQL/PostgreSQL database
- Persistent storage
- Redis for caching/sessions
- Public exposure via Ngrok
- High availability considerations

## Assignment Requirements Analysis

According to the README.md marking scheme:

| Requirement                              | Points | Status               |
| ---------------------------------------- | ------ | -------------------- |
| Use Gitea Helm chart for persistent data | 3      | ❌ Not implemented   |
| External database integration            | 3      | ❌ Not implemented   |
| Public exposure using Ngrok              | 2      | ❌ Not implemented   |
| Accurate and usable README               | 2      | ✅ Present but basic |
| **Total**                                | **10** | **2/10**             |

## Current State

### What Works

1. ✅ Basic Gitea deployment in development mode
2. ✅ Ansible orchestration structure
3. ✅ Development environment setup
4. ✅ Clean teardown process

### What Needs Implementation

1. ❌ Persistent storage configuration
2. ❌ External database setup (MySQL/PostgreSQL)
3. ❌ Production-grade Helm values
4. ❌ Ngrok integration for public access
5. ❌ Enhanced documentation

## Getting Started

### Prerequisites

```bash
pip install ansible kubernetes
```

### Quick Start

```bash
# Initialize git submodules
git submodule update --init --recursive

# Deploy the stack
ansible-playbook up.yml

# Wait for pods to be ready
kubectl get pod

# Port forward to access Gitea
kubectl port-forward svc/gitea-http 3000:3000
```

### Teardown

```bash
ansible-playbook down.yml
```

## Security Considerations

1. **Development Container**: Uses elevated security options (`SYS_PTRACE`, `seccomp=unconfined`)
2. **Secrets Management**: No current implementation for secure credential handling
3. **Network Exposure**: Local port-forwarding only (needs Ngrok for production)

## License

The project is licensed under the MIT License, attributed to Conestoga College - ACSIT (2025).

## Recommendations for Production Migration

1. **Enable Persistence**: Update `values.yaml` to enable persistent volumes
2. **Database Migration**: Configure external MySQL/PostgreSQL
3. **Security Enhancement**: Implement proper secrets management
4. **Monitoring**: Add health checks and monitoring
5. **High Availability**: Configure multi-replica deployment
6. **Public Access**: Integrate Ngrok as specified in requirements

This project serves as a foundational DevOps learning exercise, demonstrating infrastructure as code principles, container orchestration, and the transition from development to production environments.
