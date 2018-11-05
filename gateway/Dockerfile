FROM debian:jessie

RUN apt-get update && \
    apt-get install -qyy \
    -o APT::Install-Recommends=false -o APT::Install-Suggests=false \
    python3 python-pip ca-certificates libpq-dev python-psycopg2 curl netcat rlwrap telnet  build-essential python3-dev && \
    cd /usr/local/bin && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

RUN pip install virtualenv

RUN virtualenv -p python3 /appenv
RUN . /appenv/bin/activate; pip install -U pip; pip install wheel

ADD wheelhouse /var/nameko/wheelhouse

COPY config.yml /var/nameko/config.yml
COPY run.sh /var/nameko/run.sh

RUN chmod +x /var/nameko/run.sh

WORKDIR /var/nameko/

RUN . /appenv/bin/activate; \
    pip install --no-index -f wheelhouse nameko_examples_gateway

EXPOSE 8000

CMD . /appenv/bin/activate; \
    /var/nameko/run.sh;

