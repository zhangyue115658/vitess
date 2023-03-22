ARG bootstrap_version=15
ARG image="vitess/bootstrap:${bootstrap_version}-common"

FROM "${image}"

# copy apt sourse list
COPY  sources.list /etc/apt/sources.list
#RUN chmod 666 /etc/resolv.conf && echo "nameserver 114.114.114.114" >> /etc/resolv.conf
COPY  resolv.conf /etc/resolv.conf
RUN apt-key adv --keyserver keyserver.ubuntu.com  --no-tty --recv-keys 3B4FE6ACC0B21F32
RUN apt-get update 
RUN apt-get install -y sudo curl vim jq

# Install dependencies
COPY docker/lite/install_dependencies.sh /vt/dist/install_dependencies.sh
RUN bash /vt/dist/install_dependencies.sh mysql57

COPY docker/local/install_local_dependencies.sh /vt/dist/install_local_dependencies.sh
RUN bash /vt/dist/install_local_dependencies.sh
RUN echo "source /vt/common/env.sh" >> /etc/bash.bashrc

# Allows some docker builds to disable CGO
ARG CGO_ENABLED=0

# Re-copy sources from working tree.
COPY --chown=vitess:vitess . /vt/src/vitess.io/vitess

# Build and install Vitess in a temporary output directory.
USER vitess

WORKDIR /vt/src/vitess.io/vitess
RUN make install PREFIX=/vt/install

ENV VTROOT /vt/src/vitess.io/vitess
ENV VTDATAROOT /vt/vtdataroot
ENV PATH $VTROOT/bin:$PATH
ENV PATH="/var/opt/etcd:${PATH}"

RUN mkdir /vt/local
COPY examples/local /vt/local

# Copy the vtadmin web app to the correct location and npm install
COPY --chown=vitess:vitess web /web
RUN npm install /web/vtadmin

RUN mkdir /vt/common
COPY examples/common /vt/common

CMD cd /vt/local && ./101_initial_cluster.sh && /bin/bash
