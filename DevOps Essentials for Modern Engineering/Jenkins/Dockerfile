FROM jenkins/inbound-agent:latest

USER root

# -----------------------------------------------------------------------------
# 1. System Dependencies & Python 3.13 (Debian Trixie Default)
# -----------------------------------------------------------------------------
# Note: software-properties-common removed (not needed/available in Trixie)
# Note: python3.11 replaced with python3.13
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        curl \
        wget \
        git \
        unzip \
        ca-certificates \
        gnupg \
        lsb-release \
        apt-transport-https \
        jq \
        python3.13 \
        python3.13-dev \
        python3.13-venv \
        python3-pip \
    && rm -rf /var/lib/apt/lists/* /usr/share/doc /usr/share/man

# Create symlinks for python 3.13
RUN ln -sf /usr/bin/python3.13 /usr/bin/python && \
    ln -sf /usr/bin/pip3 /usr/bin/pip

# -----------------------------------------------------------------------------
# 2. Python Packages
# -----------------------------------------------------------------------------
# --break-system-packages is required for Debian 12+ (PEP 668)
RUN pip install --no-cache-dir --break-system-packages \
    boto3 \
    awscli \
    requests \
    pytest \
    black \
    flake8 \
    mypy

# -----------------------------------------------------------------------------
# 3. Docker Engine (Debian Bookworm/Trixie Compatible)
# -----------------------------------------------------------------------------
RUN curl -fsSL https://download.docker.com/linux/debian/gpg | gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg && \
    echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/debian bookworm stable" > /etc/apt/sources.list.d/docker.list && \
    apt-get update && \
    apt-get install -y --no-install-recommends docker-ce docker-ce-cli containerd.io && \
    rm -rf /var/lib/apt/lists/*

# -----------------------------------------------------------------------------
# 4. Node.js 18 LTS
# -----------------------------------------------------------------------------
RUN curl -fsSL https://deb.nodesource.com/setup_18.x | bash - && \
    apt-get install -y --no-install-recommends nodejs && \
    rm -rf /var/lib/apt/lists/*

# -----------------------------------------------------------------------------
# 5. CLI Tools: yq, kubectl, Helm, Terraform
# -----------------------------------------------------------------------------
# Install yq
RUN curl -sSLo /usr/local/bin/yq https://github.com/mikefarah/yq/releases/latest/download/yq_linux_amd64 && \
    chmod +x /usr/local/bin/yq

# Install kubectl (Pinned to v1.30.0 for stability, update as needed)
ARG KUBECTL_VERSION=v1.30.0
RUN curl -L -o /usr/local/bin/kubectl "https://dl.k8s.io/release/${KUBECTL_VERSION}/bin/linux/amd64/kubectl" && \
    chmod +x /usr/local/bin/kubectl

# Install Helm
RUN curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

# Install Terraform
RUN curl -fsSL https://apt.releases.hashicorp.com/gpg | gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg && \
    echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com bookworm main" > /etc/apt/sources.list.d/hashicorp.list && \
    apt-get update && \
    apt-get install -y --no-install-recommends terraform && \
    rm -rf /var/lib/apt/lists/*

# -----------------------------------------------------------------------------
# 6. AWS CLI v2
# -----------------------------------------------------------------------------
RUN curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip" && \
    unzip -q awscliv2.zip && \
    ./aws/install --bin-dir /usr/local/bin --install-dir /usr/local/aws-cli --update && \
    rm -rf awscliv2.zip aws/

# -----------------------------------------------------------------------------
# 7. Security & User Configuration
# -----------------------------------------------------------------------------
# Add jenkins user to docker group
RUN usermod -aG docker jenkins

# Cleanup caches
RUN rm -rf /var/cache/apt/archives/* /var/log/apt/* /root/.cache

# Switch back to non-root user
USER jenkins

# Set working directory
WORKDIR /home/jenkins/agent

# -----------------------------------------------------------------------------
# 8. Verification
# -----------------------------------------------------------------------------
RUN python --version && \
    pip --version && \
    docker --version && \
    node --version && \
    npm --version && \
    aws --version && \
    kubectl version --client && \
    helm version && \
    terraform version

ENTRYPOINT ["jenkins-agent"] 
