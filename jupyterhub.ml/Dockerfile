FROM fluxcapacitor/package-spark-2.1.0:master

# Based on the following:  https://github.com/jupyterhub/jupyterhub/blob/master/Dockerfile

# install nodejs, utf8 locale
ENV DEBIAN_FRONTEND noninteractive

RUN apt-get -y update && \
    apt-get -y upgrade && \
    apt-get -y install npm nodejs nodejs-legacy wget locales git

# libav-tools for matplotlib anim
RUN apt-get update && \
    apt-get install -y --no-install-recommends libav-tools && \
    apt-get clean
#    rm -rf /var/lib/apt/lists/*

# Install JupyterHub dependencies
RUN npm install -g configurable-http-proxy && rm -rf ~/.npm

WORKDIR /root

RUN \
  conda install --yes seaborn

ENV \
 BAZEL_VERSION=0.4.4

RUN \
  mkdir /root/bazel \
  && cd /root/bazel \
  && curl -fSsL -O https://github.com/bazelbuild/bazel/releases/download/$BAZEL_VERSION/bazel-$BAZEL_VERSION-installer-linux-x86_64.sh \
  && curl -fSsL -o /root/bazel/LICENSE.txt https://raw.githubusercontent.com/bazelbuild/bazel/master/LICENSE.txt \
  && chmod +x bazel-*.sh \
  && ./bazel-$BAZEL_VERSION-installer-linux-x86_64.sh \
  && rm bazel-$BAZEL_VERSION-installer-linux-x86_64.sh

ENV \
  TENSORFLOW_HOME=/root/tensorflow

ENV \
 TENSORFLOW_VERSION=1.0.1

#RUN \
#  git clone -b v$TENSORFLOW_VERSION https://github.com/tensorflow/tensorflow.git

#RUN \
#  cd $TENSORFLOW_HOME \
#  && printf "\n\n\n\n\n\n\n\n\n" | ./configure

#RUN \
#  cd $TENSORFLOW_HOME \
#  && bazel build --config=opt //tensorflow/tools/pip_package:build_pip_package

#RUN \
#  cd $TENSORFLOW_HOME \
#  && bazel-bin/tensorflow/tools/pip_package/build_pip_package /tmp/tensorflow_pkg

# Utility for optimizing/simplifying models for inference
#RUN \
#  cd $TENSORFLOW_HOME \
#  && bazel build tensorflow/python/tools:optimize_for_inference

#RUN \
#  pip install /tmp/tensorflow_pkg/tensorflow-${TENSORFLOW_VERSION}-cp35-cp35m-linux_x86_64.whl

#RUN \
#  echo "deb http://cran.rstudio.com/bin/linux/ubuntu trusty/" >> /etc/apt/sources.list \
#  && gpg --keyserver keyserver.ubuntu.com --recv-key E084DAB9 \
#  && gpg -a --export E084DAB9 | apt-key add - \
#  && apt-get update \
#  && apt-get install -y r-base \
#  && apt-get install -y r-base-dev

# TODO:  Replace with conda version of SparkR:
#          https://www.continuum.io/blog/developer-blog/anaconda-r-users-sparkr-and-rbokeh
#RUN \
#  apt-get install -y libcurl4-openssl-dev \
#  && apt-get install -y libzmq3 libzmq3-dev \
#  && apt-get install -y zip \
#  && ln -s /bin/tar /bin/gtar \
#  && R -e "install.packages(c('pbdZMQ','rzmq','repr', 'devtools'), type = 'source', repos = c('http://cran.us.r-project.org', 'http://irkernel.github.io/'))" \
#  && R -e "devtools::install_github('IRkernel/IRdisplay')" \
#  && R -e "devtools::install_github('IRkernel/IRkernel')" \
#  && R -e "IRkernel::installspec(user = FALSE)"

RUN \
  conda install --yes -c conda-forge jupyterhub=0.6.1 \
  && conda install --yes -c conda-forge ipykernel=4.5.0 \
  && conda install --yes -c conda-forge notebook=4.2.3 \
  && conda install --yes -c conda-forge findspark=1.0.0 \
  && conda install --yes -c conda-forge jupyter_contrib_nbextensions \
  && conda install --yes -c anaconda-nb-extensions anaconda-nb-extensions

# Install non-secure dummyauthenticator for jupyterhub (dev purposes only)
RUN \
  pip install jupyterhub-dummyauthenticator

ENV KUBERNETES_VERSION=1.5.1

RUN \
  wget https://storage.googleapis.com/kubernetes-release/release/v$KUBERNETES_VERSION/bin/linux/amd64/kubectl

RUN \
  chmod a+x kubectl \
  && mv kubectl /usr/local/bin/kubectl

RUN \
  pip install jupyterhub-simplespawner

RUN \
  mkdir -p /root/tensorboard

# lib & notesbooks provided via Kubernets PersistentVolume Mount
# from https://github.wvrgroup.internal/omni-hub/source.ml -> /root/pipeline
# .kube/config is volume mounted from Kubernetes secret

COPY config/jupyterhub/ config/jupyterhub/
COPY profiles/ /root/.ipython/

RUN \
  pip install jupyterlab \
  && jupyter serverextension enable --py jupyterlab --sys-prefix

RUN \
  pip install tensorflow==1.0.1

EXPOSE 6006 8754

COPY run run

CMD ["supervise", "."]
