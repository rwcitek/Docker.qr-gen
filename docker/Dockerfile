
FROM       ubuntu:20.04
MAINTAINER Robert Citek robert.citek@gmail.com

RUN   DEBIAN_FRONTEND=noninteractive && \
      apt-get update && \
      apt-get install -y \
        file \
        imagemagick \
        jq \
        less \
        pandoc \
        poppler-utils \
        qrencode \
        rsync \
        tidy \
        tree \
        uuid-runtime \
        vim \
        zbar-tools \
        ;

COPY Dockerfile /

