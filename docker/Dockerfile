
FROM       ubuntu:20.04
MAINTAINER Robert Citek robert.citek@gmail.com

RUN   DEBIAN_FRONTEND=noninteractive && \
      apt-get update && \
      apt-get install -y \
        qrencode \
        uuid-runtime \
        file \
        zbar-tools \
        tree \
        less \
        rsync \
        jq \
        poppler-utils \
        vim \
        imagemagick

COPY Dockerfile /

