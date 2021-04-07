FROM argoproj/argocd:v1.8.7

# Switch to root for the ability to perform install
USER root

# Install tools needed for your repo-server to retrieve & decrypt secrets, render manifests 
# (e.g. curl, awscli, gpg, sops)
RUN apt-get update && \
    apt-get install -y \
        curl \
        gpg && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* && \
    curl -o /usr/local/bin/sops -L https://github.com/mozilla/sops/releases/download/3.2.0/sops-3.2.0.linux && \
    chmod +x /usr/local/bin/sops && \
    curl -o /tmp/kustomize.tar.gz -L https://github.com/kubernetes-sigs/kustomize/releases/download/kustomize%2Fv4.0.5/kustomize_v4.0.5_linux_amd64.tar.gz && \
    tar -xzf /tmp/kustomize.tar.gz && \
    chmod +x kustomize && \
    mkdir -p /custom-tools && \
    mv kustomize /custom-tools/kustomize_4_0_5 
    
# Switch back to non-root user
USER argocd
