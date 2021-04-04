FROM ubuntu:20.04
LABEL GitLab Inc. <support@gitlab.com>

SHELL ["/bin/bash", "-c"],

# Default to supporting utf-8
ENV LANG=C.UTF-8

# Install required packages
RUN apt-get update -q \
    && DEBIAN_FRONTEND=noninteractive apt-get install -yq --no-install-recommends \
      ca-certificates \
      openssh-server \
      wget \
      vim \
      tzdata \
      nano \
      less \
    && rm -rf /var/lib/apt/lists/*

# Remove MOTD
RUN rm -rf /etc/update-motd.d /etc/motd /etc/motd.dynamic
RUN ln -fs /dev/null /run/motd.dynamic

# Copy assets
COPY RELEASE /
COPY assets/ /assets/
# COPY gitlab-ce_13.2.0-ce.0_amd64.deb /tmp/gitlab.deb
# as gitlab-ci checks out with mode 666 we need to set permissions of the files we copied into the
# container to a secure value. Issue #5956
RUN chmod -R og-w /assets RELEASE ; \
  /assets/setup

# Allow to access embedded tools
ENV PATH /opt/gitlab/embedded/bin:/opt/gitlab/bin:/assets:$PATH

# Resolve error: TERM environment variable not set.
ENV TERM xterm

# Expose web & ssh
EXPOSE 443 80 22

# Define data volumes
VOLUME ["/etc/gitlab", "/var/opt/gitlab", "/var/log/gitlab"]

# Wrapper to handle signal, trigger runit and reconfigure GitLab
CMD ["/assets/wrapper"]

HEALTHCHECK --interval=60s --timeout=30s --retries=5 \
CMD /opt/gitlab/bin/gitlab-healthcheck --fail --max-time 10

# docker run \
# --env GITLAB_OMNIBUS_CONFIG="external_url 'http://localhost:8888';" \
# --hostname localhost \
# -p 8888:80 -p 2222:22 \
# --name custom_omnibus --rm -it omnibus-gitlab-image:custom bash

# docker run --env GITLAB_OMNIBUS_CONFIG="external_url 'http://localhost:8888';" --hostname localhost -p 8888:80 -p 2222:22 --name custom_omnibus --rm omnibus-gitlab-image:custom

# docker build -t omnibus-gitlab-image:custom  -f Dockerfile .