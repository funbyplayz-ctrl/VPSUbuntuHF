# 24/7 Persistent Ubuntu VS Code Server on Hugging Face

This guide provides a step-by-step walkthrough to deploy a persistent, secure, and fully functional **Ubuntu 24.04 LTS** cloud development environment using Hugging Face Spaces and `code-server` (VS Code in the browser), kept awake 24/7 via automated pings.

---

## 🛠️ Prerequisites

* A **Hugging Face** account.
* A **cron-job.org** account (free) to keep the environment running.

---

## 📅 Step 1: Create a Private Hugging Face Space

1. Navigate to Hugging Face and click **New Space**.
2. Give your Space a name (e.g., `VPS123`).
3. Select **Docker** as the SDK.
4. Choose the **Blank** template.
5. Set the Space visibility to **Private** to protect your workspace.

---

## 🔑 Step 2: Configure Environment Secrets

Before building the container, you must set up your secure access password.

1. Inside your Space, click on the **Settings** tab at the top right.
2. Scroll down to the **Variables and secrets** section.
3. Click **New secret**.
4. Set the key name exactly as: `PASSWORD`
5. Set the value to your preferred secure password (you will use this to log into VS Code).

---

## 🐳 Step 3: Add the Custom Ubuntu Dockerfile

Create or edit the `Dockerfile` in the root of your Space repository with the following optimized Ubuntu 24.04 configuration:

```dockerfile
FROM ubuntu:24.04

# Prevent interactive prompts during installation
ENV DEBIAN_FRONTEND=noninteractive

# Update system and install base packages + Ubuntu utilities
RUN apt-get update && apt-get install -y \
    curl \
    sudo \
    git \
    wget \
    build-essential \
    python3 \
    python3-pip \
    && rm -rf /var/lib/apt/lists/*

# Install the latest version of code-server using the official Ubuntu script
RUN curl -fsSL [https://code-server.dev/install.sh](https://code-server.dev/install.sh) | sh

# Give the built-in 'ubuntu' user (UID 1000) passwordless sudo rights
RUN echo "ubuntu ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers

# Prepare safe workspace directories inside the default ubuntu profile
RUN mkdir -p /home/ubuntu/workspace && chown -R ubuntu:ubuntu /home/ubuntu

USER ubuntu
WORKDIR /home/ubuntu/workspace

# Hugging Face app port
EXPOSE 7860

# Start code-server
CMD ["code-server", "--bind-addr", "0.0.0.0:7860", "--auth", "password"]
