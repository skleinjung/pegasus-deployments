# [Choice] Python version: 3, 3.9, 3.8, 3.7, 3.6
ARG VARIANT=3.9
FROM mcr.microsoft.com/vscode/devcontainers/python:${VARIANT}

# Suppress an apt-key warning about standard out not being a terminal. Use in this script is safe.
ENV APT_KEY_DONT_WARN_ON_DANGEROUS_USAGE=DontWarn
ARG USERNAME=sean

RUN userdel vscode \
  && useradd --create-home --shell /bin/bash --uid 1000 --no-log-init sean

# install gcloud cli
# RUN echo "deb [signed-by=/usr/share/keyrings/cloud.google.gpg] https://packages.cloud.google.com/apt cloud-sdk main" \
#     | sudo tee -a /etc/apt/sources.list.d/google-cloud-sdk.list && \
#   sudo apt-get -y install \
#     apt-transport-https \
#     ca-certificates \
#     gnupg && \
#   curl https://packages.cloud.google.com/apt/doc/apt-key.gpg \
#     | sudo apt-key --keyring /usr/share/keyrings/cloud.google.gpg add - && \
#   sudo apt-get update && sudo apt-get install -y \
#     google-cloud-sdk \
#     google-cloud-sdk-pubsub-emulator && \
#   rm -rf /var/lib/apt/lists/*

# [Optional] If your pip requirements rarely change, uncomment this section to add them to the image.
COPY requirements.txt /tmp/pip-tmp/
RUN pip3 --disable-pip-version-check --no-cache-dir install -r /tmp/pip-tmp/requirements.txt \
   && rm -rf /tmp/pip-tmp

# install arkade and useful tools
RUN curl -sLS https://dl.get-arkade.dev | sh \
  && arkade get helm \
  && arkade get kubectl \
  && arkade get kubeseal \
  && arkade get kustomize \
  && arkade get terraform \
  && arkade get yq \
  && chmod +x /root/.arkade/bin/* \
  && mv /root/.arkade/bin/* /usr/local/bin/

# install latest ArgoCD cli
RUN VERSION=$(curl --silent "https://api.github.com/repos/argoproj/argo-cd/releases/latest" | grep '"tag_name"' | sed -E 's/.*"([^"]+)".*/\1/') \
  && curl -sSL -o /usr/local/bin/argocd https://github.com/argoproj/argo-cd/releases/download/$VERSION/argocd-linux-amd64 \
  && chmod +x /usr/local/bin/argocd

# prepare postgres-operator client
# must run 'pgo-setup.sh' after configuring kubectl for a cluster
RUN curl https://raw.githubusercontent.com/CrunchyData/postgres-operator/v4.6.0/installers/kubectl/client-setup.sh > /usr/local/bin/pgo-setup.sh \
  && chmod +x /usr/local/bin/pgo-setup.sh \
  && echo "export PGOUSER='/home/$USERNAME/.pgo/pgo/pgouser'" >> /home/$USERNAME/.bashrc \
  && echo "export PGO_CA_CERT='/home/$USERNAME/.pgo/pgo/client.crt'" >> /home/$USERNAME/.bashrc \
  && echo "export PGO_CLIENT_CERT='/home/$USERNAME/.pgo/pgo/client.crt'" >> /home/$USERNAME/.bashrc \
  && echo "export PGO_CLIENT_KEY='/home/$USERNAME/.pgo/pgo/client.key'" >> /home/$USERNAME/.bashrc \
  && echo "export PGO_APISERVER_URL='https://127.0.0.1:8443'" >> /home/$USERNAME/.bashrc \
  && echo "export PGO_NAMESPACE=pgo" >> /home/$USERNAME/.bashrc \
  && echo "export PATH=/home/$USERNAME/.pgo/pgo:$PATH" >> /home/$USERNAME/.bashrc
  
# install latest terraform (newer than arkade's)
# RUN curl -fsSL https://apt.releases.hashicorp.com/gpg | apt-key add - && \
#   apt-get update && \
#   apt-get install -y software-properties-common && \
#   apt-add-repository "deb [arch=amd64] https://apt.releases.hashicorp.com $(lsb_release -cs) main" && \
#   apt-get update && \
#   apt-get install -y terraform && \
#   rm -rf /var/lib/apt/lists/*

RUN SNIPPET="export PROMPT_COMMAND='history -a' && export HISTFILE=/home/$USERNAME/scratch/.bash_history" \
    && echo $SNIPPET >> "/home/$USERNAME/.bashrc" \
    && echo "source ~/scratch/.bashrc" >> "/home/$USERNAME/.bashrc"

# [Option] Install Node.js
# ARG INSTALL_NODE="true"
# ARG NODE_VERSION="lts/*"
# RUN if [ "${INSTALL_NODE}" = "true" ]; then su sean -c "umask 0002 && . /usr/local/share/nvm/nvm.sh && nvm install ${NODE_VERSION} 2>&1"; fi

# [Optional] Uncomment this section to install additional OS packages.
# RUN apt-get update && export DEBIAN_FRONTEND=noninteractive \
#     && apt-get -y install --no-install-recommends <your-package-list-here>

# [Optional] Uncomment this line to install global node packages.
# RUN su sean -c "source /usr/local/share/nvm/nvm.sh && npm install -g <your-package-here>" 2>&1

USER sean