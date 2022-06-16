FROM ubuntu:latest
RUN apt-get update && apt-get install -y openssh-server sudo curl
RUN useradd --create-home --shell /bin/bash carv && echo "carv:carv" | chpasswd && adduser carv sudo && mkdir -p /home/carv/.ssh
# COPY --chown=carv:carv container_id /home/carv/.ssh/id_rsa
# COPY --chown=carv:carv container_id.pub /home/carv/.ssh/id_rsa.pub

ENV APISERVER_CERT_LOCATION /vkubelet-knoc-0-crt.pem
ENV APISERVER_KEY_LOCATION /vkubelet-knoc-0-key.pem
ENV KUBELET_PORT 10250

# Copy the configuration file for the knoc provider.
COPY deploy/vkubelet-knoc-0-cfg.json /vkubelet-knoc-0-cfg.json
# Copy the certificate for the HTTPS server.
COPY deploy/vkubelet-knoc-0-crt.pem /vkubelet-knoc-0-crt.pem
# Copy the private key for the HTTPS server.
COPY deploy/vkubelet-knoc-0-key.pem /vkubelet-knoc-0-key.pem

COPY bin/virtual-kubelet /virtual-kubelet
COPY bin/door /usr/bin/door

RUN curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
RUN chmod +x kubectl && \
    mv ./kubectl /usr/local/bin/kubectl


USER carv
CMD ["/virtual-kubelet"]
