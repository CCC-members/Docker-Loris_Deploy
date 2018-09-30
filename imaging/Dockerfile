FROM loris-app
LABEL maintainer="mario462@gmail.com"

#Directories setup
ARG projectname=loris
RUN mkdir -p /data/${projectname}/bin/mri && \
    chown -R lorisadmin:lorisadmin /data/${projectname}
#Clone Loris-MRI project
RUN git clone -b master https://github.com/aces/Loris-MRI.git /data/${projectname}/bin/mri
WORKDIR /data/${projectname}/bin/mri
#Install necessary software
RUN apt-get update && \
    apt-get install -yy --no-install-recommends libc6 libstdc++6 imagemagick perl octave libgl1-mesa-glx libglu1-mesa && \
    rm -rf /var/lib/apt/lists/*
#Download and install binaries
ARG url=http://packages.bic.mni.mcgill.ca/minc-toolkit/Debian/
ARG beast=beast-library-1.1.0-20121212.deb
ARG bic=bic-mni-models-0.1.1-20120421.deb
ARG testsuite=minc-toolkit-testsuite-0.1.3-20131212.deb
ARG toolkit=minc-toolkit-1.9.16-20180117-Debian_8.7-x86_64.deb
ARG toolkitversion=1.9.16
RUN wget ${url}${beast} && \
    wget ${url}${bic} && \
    wget ${url}${testsuite} && \
    wget ${url}${toolkit}
RUN dpkg -i ${toolkit} ${testsuite} ${bic} ${beast} && \
    apt-get install -f
#Remove unnecessary .deb files
RUN rm ${toolkit} ${testsuite} ${bic} ${beast}
#Source toolkit environment
RUN echo "source /opt/minc/${toolkitversion}/minc-toolkit-config.sh" >> /home/lorisadmin/.bashrc && \
    echo "source /data/${projectname}/bin/mri/environment" >> /home/lorisadmin/.bashrc && \
    echo "source /opt/minc/${toolkitversion}/minc-toolkit-config.sh" >> /etc/bash.bashrc && \
    echo "source /data/${projectname}/bin/mri/environment" >> /etc/bash.bashrc


#TODO MERGE WITH ABOVE INSTALLS
ENV DEBIAN_FRONTEND noninteractive
RUN apt-get update && \
    apt-get install -yy --no-install-recommends mysql-client && \
    rm -rf /var/lib/apt/lists/*

