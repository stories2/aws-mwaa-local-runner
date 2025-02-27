# VERSION 2.4
# AUTHOR: John Jackson
# DESCRIPTION: Amazon MWAA Local Dev Environment
# BUILD: docker build --rm -t amazon/mwaa-local .

FROM amazonlinux:2.0.20230307.0
LABEL maintainer="amazon"

# Airflow
## Version specific ARGs
ARG AIRFLOW_VERSION=2.4.3
ARG WATCHTOWER_VERSION=2.0.1
ARG PROVIDER_AMAZON_VERSION=6.0.0

## General ARGs
ARG AIRFLOW_USER_HOME=/usr/local/airflow
ARG AIRFLOW_DEPS=""
ARG PYTHON_DEPS=""
ARG SYSTEM_DEPS=""
ARG INDEX_URL=""
ENV AIRFLOW_HOME=${AIRFLOW_USER_HOME}
ENV PATH="$PATH:/usr/local/airflow/.local/bin:/root/.local/bin:/usr/local/airflow/.local/lib/python3.10/site-packages"
ENV PYTHON_VERSION=3.10.9

COPY script/bootstrap.sh /bootstrap.sh
COPY script/systemlibs.sh /systemlibs.sh
COPY script/generate_key.sh /generate_key.sh
COPY config/constraints.txt /constraints.txt
COPY config/mwaa-base-providers-requirements.txt /mwaa-base-providers-requirements.txt

RUN chmod u+x /systemlibs.sh && /systemlibs.sh
RUN chmod u+x /bootstrap.sh && /bootstrap.sh
RUN chmod u+x /generate_key.sh && /generate_key.sh

# Post bootstrap to avoid expensive docker rebuilds
COPY script/entrypoint.sh /entrypoint.sh
COPY config/airflow.cfg ${AIRFLOW_USER_HOME}/airflow.cfg
COPY config/webserver_config.py ${AIRFLOW_USER_HOME}/webserver_config.py

RUN chown -R airflow: ${AIRFLOW_USER_HOME}
RUN chmod +x /entrypoint.sh

RUN wget https://sqlite.org/2023/sqlite-autoconf-3410100.tar.gz
RUN tar -xvf sqlite-autoconf-3410100.tar.gz
WORKDIR sqlite-autoconf-3410100
RUN ./configure
RUN make clean && make -j 20 && make install
WORKDIR ..
ENV LD_LIBRARY_PATH="/usr/local/lib"
RUN source /etc/profile

EXPOSE 8080 5555 8793

USER airflow
WORKDIR ${AIRFLOW_USER_HOME}
ENTRYPOINT ["/entrypoint.sh"]
CMD ["local-runner"]
