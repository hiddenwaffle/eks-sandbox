FROM amazon/aws-cli

WORKDIR /work
ENTRYPOINT ["/usr/bin/env", "bash"]

RUN yum install -y jq
COPY kubectl-v1.20.4 /usr/local/bin/kubectl
