# This Dockerfile is used to build an ROS + VNC + Tensorflow image based on Ubuntu 18.04
FROM nvcr.io/nvidia/pytorch:21.10-py3

LABEL maintainer "SheldonFung"
MAINTAINER SheldonFung "https://github.com/Sheldonvon98"
ENV REFRESHED_AT 2022-1-11

ENV TZ=Australia/Melbourne
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone
ENV DEBIAN_FRONTEND noninteractive
# Install sudo
RUN apt-get update && \
    apt-get install -y sudo \
    xterm \
    curl \
    wget \
    meld \
    && apt-get clean && rm -rf /usr/local/src/* &&  rm -rf /tmp/* /var/tmp/* $HOME/.cache/* /var/cache/apt/*

# Configure user
ARG user=XQLab
ARG passwd=asdasdasd
ARG uid=1000
ARG gid=1000
ENV USER=$user
ENV PASSWD=$passwd
ENV UID=$uid
ENV GID=$gid
RUN groupadd $USER && \
    useradd --create-home --no-log-init -g $USER $USER && \
    usermod -aG sudo $USER && \
    echo "$USER:$PASSWD" | chpasswd && \
    chsh -s /bin/bash $USER && \
    # Replace 1000 with your user/group id
    usermod  --uid $UID $USER && \
    groupmod --gid $GID $USER

### Install VScode
RUN cd /home/$USER/ &&\
    # Tmp fix to run vs code without no-sandbox: https://github.com/microsoft/vscode/issues/126027
    wget -q https://az764295.vo.msecnd.net/stable/054a9295330880ed74ceaedda236253b4f39a335/code_1.56.2-1620838498_amd64.deb -O ./vscode.deb &&\
    # wget -q https://go.microsoft.com/fwlink/?LinkID=760868 -O ./vscode.deb &&\
    sudo apt-get update &&\
    sudo apt-get install -y ./vscode.deb &&\
    sudo rm ./vscode.deb &&\
    sudo rm /etc/apt/sources.list.d/vscode.list &&\
    sudo apt-get clean && sudo rm -rf /usr/local/src/* && sudo rm -rf /tmp/* /var/tmp/* $HOME/.cache/* /var/cache/apt/* 


RUN sudo apt-get update && \
    sudo apt-get install -y apt-transport-https && \
    sudo apt-get install -y fonts-wqy-microhei ttf-wqy-zenhei &&\
    sudo apt-get clean && sudo rm -rf /usr/local/src/* && sudo rm -rf /tmp/* /var/tmp/* $HOME/.cache/* /var/cache/apt/* 

### VNC Installation
LABEL io.k8s.description="VNC Container with ROS with Xfce window manager" \
      io.k8s.display-name="VNC Container with ROS based on Ubuntu" \
      io.openshift.expose-services="6901:http,5901:xvnc,6006:tnesorboard" \
      io.openshift.tags="vnc, ros, gazebo, tensorflow, ubuntu, xfce" \
      io.openshift.non-scalable=true

## Connection ports for controlling the UI:
# VNC port:5901
# noVNC webport, connect via http://IP:6901/?password=vncpassword
ENV DISPLAY=:1 \
    VNC_PORT=5901 \
    NO_VNC_PORT=6901
EXPOSE $VNC_PORT $NO_VNC_PORT

## Envrionment config
ENV VNCPASSWD=vncpassword
ENV HOME=/home/$USER \
    TERM=xterm \
    STARTUPDIR=/dockerstartup \
    INST_SCRIPTS=/home/$USER/install \
    NO_VNC_HOME=/home/$USER/noVNC \
    DEBIAN_FRONTEND=noninteractive \
    VNC_COL_DEPTH=24 \
    VNC_RESOLUTION=1600x900 \
    VNC_PW=$VNCPASSWD \
    VNC_VIEW_ONLY=false
WORKDIR $HOME

## Add all install scripts for further steps
ADD ./src/common/install/ $INST_SCRIPTS/
ADD ./src/ubuntu/install/ $INST_SCRIPTS/
ADD ./src/common/novnc/ $INST_SCRIPTS/
RUN find $INST_SCRIPTS -name '*.sh' -exec chmod a+x {} +

## Install some common tools
RUN $INST_SCRIPTS/tools.sh
ENV LANG='en_US.UTF-8' LANGUAGE='en_US:en' LC_ALL='en_US.UTF-8'

## Install xvnc-server & noVNC - HTML5 based VNC viewer
RUN $INST_SCRIPTS/tigervnc.sh
RUN $INST_SCRIPTS/no_vnc.sh

## Install firefox and chrome browser
RUN $INST_SCRIPTS/firefox.sh
RUN $INST_SCRIPTS/chrome.sh

## Install xfce UI
RUN $INST_SCRIPTS/xfce_ui.sh
ADD ./src/common/xfce/ $HOME/

## configure startup
RUN $INST_SCRIPTS/libnss_wrapper.sh
ADD ./src/common/scripts $STARTUPDIR
RUN $INST_SCRIPTS/set_user_permission.sh $STARTUPDIR $HOME

# prepare default python 2.7 environment
USER root
#RUN pip install --ignore-installed --no-cache-dir --upgrade https://storage.googleapis.com/tensorflow/linux/gpu/tensorflow_gpu-1.11.0-cp27-none-linux_x86_64.whl && \
#    pip install --no-cache-dir keras==2.2.4 matplotlib pandas scipy h5py testresources scikit-learn

# Expose Tensorboard
EXPOSE 6006

# Expose Jupyter 
EXPOSE 8888

### Switch to root user to install additional software
USER $USER

ENTRYPOINT ["/dockerstartup/vnc_startup.sh"]
CMD ["--wait"]