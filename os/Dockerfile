FROM ubuntu:latest
RUN apt update
RUN apt install -y wget vim openjdk-8-jdk build-essential zlib1g-dev libssl-dev libbz2-dev \
    libreadline-dev libsqlite3-dev libncurses5-dev libgdbm-dev \
    libdb-dev libexpat1-dev liblzma-dev tk-dev libffi-dev

# install python2.y
RUN wget https://www.python.org/ftp/python/2.7.18/Python-2.7.18.tgz
RUN tar -xzf Python-2.7.18.tgz
RUN cd Python-2.7.18 && ./configure --with-openssl && make -j$(nproc) && make install


# install Cassandra 3.0
RUN wget https://downloads.apache.org/cassandra/3.0.30/apache-cassandra-3.0.30-bin.tar.gz \
    && tar -zxvf apache-cassandra-3.0.30-bin.tar.gz \
    && mv apache-cassandra-3.0.30 /opt/cassandra \
    && rm apache-cassandra-3.0.30-bin.tar.gz

# set env
ENV CASSANDRA_HOME /opt/cassandra
ENV PATH $CASSANDRA_HOME/bin:$PATH

# define workdir
WORKDIR /opt/cassandra

RUN  mkdir -p /opt/cassandra/data/data
RUN  mkdir -p /opt/cassandra/data/commitlog
RUN  mkdir -p /opt/cassandra/data/saved_caches
RUN  mkdir -p /opt/cassandra/data/hints

EXPOSE 9042 7000 7001 7199 9160

# start Cassandra
CMD ["/bin/bash"]