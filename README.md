# CPS Jupyter Notebook

<div align="center">

[![Docker Multi-Variant Build](https://github.com/bjoernellens1/cps-jupyter-notebook/actions/workflows/docker-publish.yml/badge.svg)](https://github.com/bjoernellens1/cps-jupyter-notebook/actions/workflows/docker-publish.yml)
[![License](https://img.shields.io/github/license/bjoernellens1/cps-jupyter-notebook)](LICENSE)
[![GitHub Stars](https://img.shields.io/github/stars/bjoernellens1/cps-jupyter-notebook?style=social)](https://github.com/bjoernellens1/cps-jupyter-notebook/stargazers)

**A production-ready, multi-variant JupyterLab environment tailored for Cyber-Physical Systems development**

[Features](#-features) • [Quick Start](#-quick-start) • [Variants](#-image-variants) • [Documentation](#-documentation) • [Contributing](#-contributing)

</div>

---

## 📋 Overview

The **CPS Jupyter Notebook** project provides containerized JupyterLab environments optimized for data science, machine learning, and cyber-physical systems development. It offers two specialized variants to suit different computational needs:

- **CPU-only variant**: Lightweight, multi-architecture support (amd64/arm64)
- **CUDA GPU variant**: NVIDIA GPU-accelerated for deep learning workloads

Built on trusted upstream images and extended with essential development tools, these images are designed for both individual developers and team environments with built-in user management, integrated development tools, and system monitoring capabilities.

### 🎯 Primary Use Case

These images are primarily used as the base notebook environment for the **[CPS JupyterHub](https://github.com/mul-cps/cps-jupyterhub-internal)** deployment, providing a consistent, feature-rich development environment for multi-user educational and research settings. While optimized for JupyterHub, they work equally well as standalone containers for individual use.

---

## ✨ Features

### 🔧 Core Capabilities

- **JupyterLab 4.2.1+** - Modern, extensible notebook interface
- **Multi-platform Support** - CPU variant runs on both x86_64 and ARM64 architectures
- **GPU Acceleration** - CUDA 12.6 support for machine learning and deep learning
- **Integrated Development** - VS Code Server integration for a complete IDE experience
- **Version Control** - Built-in Git support with GitHub CLI and credential helpers

### 📊 Monitoring & Resource Management

- **System Monitoring** - Glances and btop for real-time performance insights
- **Resource Usage** - JupyterLab extension for notebook resource tracking
- **GPU Monitoring** - NVIDIA Dashboard extension (GPU variant only)

### 🎨 Enhanced User Experience

- **Modern Themes** - Catppuccin and Horizon themes for comfortable coding
- **Language Server Protocol** - Code intelligence with Python LSP (GPU variant)
- **NBGitPuller** - Simplified repository distribution for educational environments
- **SSH & Proxy Support** - Remote access and service proxying capabilities

### 👥 User Management

- **`jovyan`** (UID 1000) - Default JupyterLab user for notebook operations
- **`cpsadmin`** - Administrative user with sudo privileges for system management

---

## 🚀 Quick Start

### Prerequisites

- **Docker** 20.10+ or **Podman** 3.0+
- **NVIDIA Container Toolkit** (for GPU variant)
- 4GB+ available RAM
- 10GB+ available disk space

### Running with Docker

#### CPU-only Variant

```bash
docker run -p 8888:8888 \
  -v "${PWD}/work:/home/jovyan/work" \
  ghcr.io/bjoernellens1/cps-jupyter-notebook:latest-cpu
```

#### GPU Variant

```bash
docker run --gpus all -p 8888:8888 \
  -v "${PWD}/work:/home/jovyan/work" \
  ghcr.io/bjoernellens1/cps-jupyter-notebook:latest-cuda-12.6
```

### Running with Docker Compose

Create a `docker-compose.yml` file:

```yaml
version: '3.8'

services:
  jupyter:
    image: ghcr.io/bjoernellens1/cps-jupyter-notebook:latest-cpu
    ports:
      - "8888:8888"
    volumes:
      - ./work:/home/jovyan/work
      - jupyter-config:/home/jovyan/.jupyter
    environment:
      - JUPYTER_ENABLE_LAB=yes
      - GRANT_SUDO=yes
    restart: unless-stopped

volumes:
  jupyter-config:
```

For GPU support, add the following to the service configuration:

```yaml
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              count: all
              capabilities: [gpu]
```

Start the service:

```bash
docker compose up -d
```

### Accessing JupyterLab

After starting the container, look for the access URL in the logs:

```bash
docker logs <container-name>
```

You'll see output like:
```
[I 2025-10-30 12:00:00.000 ServerApp] http://127.0.0.1:8888/lab?token=<your-token>
```

Open this URL in your browser to access JupyterLab.

---

## 📦 Image Variants

### CPU-only (`-cpu` suffix)

**Base:** `quay.io/jupyter/datascience-notebook:lab-4.2.1`

| Feature | Support |
|---------|---------|
| **Platforms** | `linux/amd64`, `linux/arm64` |
| **GPU Support** | ❌ No |
| **Use Cases** | General data science, education, lightweight ML |
| **Image Size** | ~4-5 GB |

**Available Tags:**
```
ghcr.io/bjoernellens1/cps-jupyter-notebook:latest-cpu
ghcr.io/bjoernellens1/cps-jupyter-notebook:main-cpu
ghcr.io/bjoernellens1/cps-jupyter-notebook:v1.0.0-cpu
ghcr.io/bjoernellens1/cps-jupyter-notebook:sha-<commit>-cpu
```

### CUDA GPU (`-cuda-12.6` suffix)

**Base:** `cschranz/gpu-jupyter:v1.9_cuda-12.6_ubuntu-24.04_slim`

| Feature | Support |
|---------|---------|
| **Platforms** | `linux/amd64` |
| **GPU Support** | ✅ NVIDIA CUDA 12.6+ |
| **Use Cases** | Deep learning, GPU-accelerated computing, large-scale ML |
| **Image Size** | ~6-8 GB |
| **Additional Features** | jupyterlab-nvdashboard, Dask, Python LSP |

**Available Tags:**
```
ghcr.io/bjoernellens1/cps-jupyter-notebook:latest-cuda-12.6
ghcr.io/bjoernellens1/cps-jupyter-notebook:main-cuda-12.6
ghcr.io/bjoernellens1/cps-jupyter-notebook:v1.0.0-cuda-12.6
ghcr.io/bjoernellens1/cps-jupyter-notebook:sha-<commit>-cuda-12.6
```

---

## 🔐 Security & User Configuration

### Setting User Passwords

Passwords are configured at build time using GitHub Actions secrets. For local builds:

```bash
# CPU variant
docker build \
  --secret id=admin_password,src=admin_pass.txt \
  --secret id=root_password,src=root_pass.txt \
  -f docker/Dockerfile.cpu \
  -t cps-jupyter:cpu ./docker

# GPU variant
docker build \
  --secret id=admin_password,src=admin_pass.txt \
  --secret id=root_password,src=root_pass.txt \
  -f docker/Dockerfile.gpu \
  -t cps-jupyter:cuda ./docker
```

### User Accounts

| User | UID | Sudo | Purpose |
|------|-----|------|---------|
| `jovyan` | 1000 | No | Default notebook user |
| `cpsadmin` | Variable | Yes | System administration |

### Switching Users

From within a terminal in JupyterLab:

```bash
# Switch to admin user
su - cpsadmin

# Return to jovyan
exit
```

---

## 🛠️ Development

### Building Locally

```bash
# Clone the repository
git clone https://github.com/bjoernellens1/cps-jupyter-notebook.git
cd cps-jupyter-notebook

# Build CPU variant
docker build -f docker/Dockerfile.cpu -t cps-jupyter:cpu ./docker

# Build GPU variant
docker build -f docker/Dockerfile.gpu -t cps-jupyter:cuda ./docker
```

### Customizing the Image

To add additional packages, modify the respective Dockerfile:

**Python packages:**
```dockerfile
RUN pip install --no-cache-dir \
    your-package-here \
    another-package
```

**System packages:**
```dockerfile
USER root
RUN apt-get update && apt-get install -y \
    your-package-here && \
    apt-get clean && rm -rf /var/lib/apt/lists/*
USER jovyan
```

### Testing Changes

```bash
# Build your modified image
docker build -f docker/Dockerfile.cpu -t cps-jupyter:test ./docker

# Run with test data
docker run -p 8888:8888 -v ./test-notebooks:/home/jovyan/work cps-jupyter:test
```

---

## 🔄 CI/CD Pipeline

The project uses GitHub Actions for automated building, testing, and publishing:

- **Multi-variant Matrix Build** - Parallel builds for CPU and GPU variants
- **Multi-platform Support** - Automatic cross-compilation for amd64/arm64 (CPU variant)
- **Smart Caching** - Separate cache scopes per variant for optimal performance
- **Automatic Tagging** - Semantic versioning, branch names, PR numbers, and commit SHAs
- **Image Signing** - Cosign signatures for supply chain security
- **Automated Deployment** - Push to GitHub Container Registry on merge to main

### Triggered By

- **Push to main** - Builds and publishes `latest` and `main` tags
- **Pull Requests** - Builds for testing without publishing
- **Version Tags** - Publishes semantic version tags (e.g., `v1.2.3`)

---

## 📚 Documentation

- **[Docker Setup Guide](docker/README.md)** - Detailed variant information and configuration
- **[JupyterLab Documentation](https://jupyterlab.readthedocs.io/)** - Official JupyterLab docs
- **[CUDA Toolkit Documentation](https://docs.nvidia.com/cuda/)** - NVIDIA CUDA reference

### Included Tools & Extensions

| Tool | CPU | GPU | Description |
|------|-----|-----|-------------|
| JupyterLab | ✅ | ✅ | Modern notebook interface |
| Git Integration | ✅ | ✅ | Version control in JupyterLab |
| VS Code Server | ✅ | ✅ | Full IDE in browser |
| GitHub CLI (`gh`) | ✅ | ✅ | GitHub operations from terminal |
| btop | ✅ | ✅ | Interactive process viewer |
| Glances | ✅ | ✅ | System monitoring tool |
| SSH Server | ✅ | ✅ | Remote terminal access |
| Python LSP | ❌ | ✅ | Code intelligence |
| Dask | ❌ | ✅ | Parallel computing |
| NVIDIA Dashboard | ❌ | ✅ | GPU monitoring |

---

## 🤝 Contributing

Contributions are welcome! Here's how you can help:

1. **Fork the repository**
2. **Create a feature branch** (`git checkout -b feature/amazing-feature`)
3. **Make your changes**
4. **Test thoroughly**
5. **Commit your changes** (`git commit -m 'Add amazing feature'`)
6. **Push to the branch** (`git push origin feature/amazing-feature`)
7. **Open a Pull Request**

### Contribution Guidelines

- Follow existing code style and conventions
- Update documentation for new features
- Test both CPU and GPU variants when applicable
- Keep Docker images as small as possible
- Document any new dependencies

---

## 🐛 Troubleshooting

### Common Issues

**JupyterLab won't start:**
```bash
# Check container logs
docker logs <container-name>

# Ensure port 8888 is available
sudo lsof -i :8888
```

**GPU not detected (GPU variant):**
```bash
# Verify nvidia-container-toolkit is installed
docker run --rm --gpus all nvidia/cuda:12.6.0-base-ubuntu24.04 nvidia-smi

# Check NVIDIA driver
nvidia-smi
```

**Permission issues:**
```bash
# Ensure work directory has correct permissions
sudo chown -R 1000:1000 ./work
```

**Out of memory:**
```bash
# Limit memory usage
docker run -m 4g -p 8888:8888 ghcr.io/bjoernellens1/cps-jupyter-notebook:latest-cpu
```

---

## 📄 License

This project is licensed under the terms specified in the [LICENSE](LICENSE) file.

---

## 🙏 Acknowledgments

This project builds upon the excellent work of:

- [Jupyter Project](https://jupyter.org/) - Core notebook technology
- [Jupyter Docker Stacks](https://github.com/jupyter/docker-stacks) - Base images
- [GPU Jupyter](https://github.com/iot-salzburg/gpu-jupyter) - GPU-enabled base image
- The broader open-source community

---

## 📞 Support

- **Issues:** [GitHub Issues](https://github.com/bjoernellens1/cps-jupyter-notebook/issues)
- **Discussions:** [GitHub Discussions](https://github.com/bjoernellens1/cps-jupyter-notebook/discussions)
- **Repository:** [bjoernellens1/cps-jupyter-notebook](https://github.com/bjoernellens1/cps-jupyter-notebook)

---

<div align="center">

**Built with ❤️ for the Cyber-Physical Systems community**

[⬆ Back to Top](#cps-jupyter-notebook)

</div>
