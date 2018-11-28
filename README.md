# Nameko Examples

https://gitlab.com/adavarski/nameko-microservices-examples-circleci-travisci

[![CircleCI](https://circleci.com/gh/adavarski/nameko-microservices-examples-circleci-travisci/tree/master.svg?style=svg)](https://circleci.com/gh/adavarski/nameko-microservices-examples-circleci-travisci)

https://gitlab.com/adavarski/nameko-microservices-examples-circleci-travisci/pipelines

## Prerequisites

* [Python 3](https://www.python.org/downloads/)
* [Docker](https://www.docker.com/)
* [Docker Compose](https://docs.docker.com/compose/)

## Overview

### Repository structure
When developing Nameko services you have the freedom to organize your repo structure any way you want.

For this example we placed 3 Nameko services: `Products`, `Orders` and `Gateway` in one repository.

While possible, this is not necessarily the best practice. Aim to apply Domain Driven Design concepts and try to place only services that belong to the same bounded context in one repository e.g., Product (main service responsible for serving products) and Product Indexer (a service responsible for listening for product change events and indexing product data within search database).

### Services

![Services](diagram.png)

#### Products Service

Responsible for storing and managing product information and exposing RPC Api that can be consumed by other services. This service is using Redis as it's data store. Example includes implementation of Nameko's [DependencyProvider](https://nameko.readthedocs.io/en/stable/key_concepts.html#dependency-injection) `Storage` which is used for talking to Redis.

#### Orders Service

Responsible for storing and managing orders information and exposing RPC Api that can be consumed by other services.

This service is using PostgreSQL database to persist order information.
- [nameko-sqlalchemy](https://pypi.python.org/pypi/nameko-sqlalchemy)  dependency is used to expose [SQLAlchemy](http://www.sqlalchemy.org/) session to the service class.
- [Alembic](https://pypi.python.org/pypi/alembic) is used for database migrations.

#### Gateway Service

Is a service exposing HTTP Api to be used by external clients e.g., Web and Mobile Apps. It coordinates all incoming requests and composes responses based on data from underlying domain services.

[Marshmallow](https://pypi.python.org/pypi/marshmallow) is used for validating, serializing and deserializing complex Python objects to JSON and vice versa in all services.

## Running examples

Quickest way to try out examples is to run them with Docker Compose

`$ docker-compose up`

Docker images for [RabbitMQ](https://hub.docker.com/_/rabbitmq/), [PostgreSQL](https://hub.docker.com/_/postgres/) and [Redis](https://hub.docker.com/_/redis/) will be automatically downloaded and their containers linked to example service containers.

When you see `Connected to amqp:...` it means services are up and running.

Gateway service with HTTP Api is listening on port 8003 and these endpoitns are available to play with:

#### Create Product

```sh
$ curl -XPOST -d '{"id": "the_odyssey", "title": "The Odyssey", "passenger_capacity": 101, "maximum_speed": 5, "in_stock": 10}' 'http://localhost:8003/products'
```

#### Get Product

```sh
$ curl 'http://localhost:8003/products/the_odyssey'

{
  "id": "the_odyssey",
  "title": "The Odyssey",
  "passenger_capacity": 101,
  "maximum_speed": 5,
  "in_stock": 10
}
```
#### Create Order

```sh
$ curl -XPOST -d '{"order_details": [{"product_id": "the_odyssey", "price": "100000.99", "quantity": 1}]}' 'http://localhost:8003/orders'

{"id": 1}
```

#### Get Order

```sh
$ curl 'http://localhost:8003/orders/1'

{
  "id": 1,
  "order_details": [
    {
      "id": 1,
      "quantity": 1,
      "product_id": "the_odyssey",
      "image": "http://www.example.com/airship/images/the_odyssey.jpg",
      "price": "100000.99",
      "product": {
        "maximum_speed": 5,
        "id": "the_odyssey",
        "title": "The Odyssey",
        "passenger_capacity": 101,
        "in_stock": 9
      }
    }
  ]
}
```

## Running tests

Ensure RabbitMQ, PostgreSQL and Redis are running and `config.yaml` files for each service are configured correctly.

`$ make coverage`

```
$ cat gateway/Dockerfile
FROM debian:jessie

RUN apt-get update && \
    apt-get install -qyy \
    -o APT::Install-Recommends=false -o APT::Install-Suggests=false \
    python3 python-pip ca-certificates libpq-dev python-psycopg2 curl netcat rlwrap telnet  build-essential python3-dev && \
    cd /usr/local/bin && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

ADD . /tmp/wheel_build

RUN pip install virtualenv

RUN virtualenv -p python3 /appenv
RUN . /appenv/bin/activate; pip install -U pip; pip install wheel; cd /tmp/wheel_build; pip wheel -w /tmp/wheel_build/wheelhouse .; mkdir -p /var/nameko/wheelhouse; cp /tmp/wheel_build/wheelhouse/* /var/nameko/wheelhouse; rm -rf /tmp/wheel_build

COPY config.yml /var/nameko/config.yml
COPY run.sh /var/nameko/run.sh

RUN chmod +x /var/nameko/run.sh

WORKDIR /var/nameko/

RUN . /appenv/bin/activate; \
    pip install --no-index -f wheelhouse nameko_examples_gateway

EXPOSE 8000

CMD . /appenv/bin/activate; \
    /var/nameko/run.sh;

$ cat orders/Dockerfile 
FROM debian:jessie

RUN apt-get update && \
    apt-get install -qyy \
    -o APT::Install-Recommends=false -o APT::Install-Suggests=false \
    python3 python-pip ca-certificates libpq-dev python-psycopg2 curl netcat rlwrap telnet  build-essential python3-dev && \
    cd /usr/local/bin && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

ADD . /tmp/wheel_build

RUN pip install virtualenv

RUN virtualenv -p python3 /appenv
RUN . /appenv/bin/activate; pip install -U pip; pip install wheel; cd /tmp/wheel_build; pip wheel -w /tmp/wheel_build/wheelhouse .; mkdir -p /var/nameko/wheelhouse; cp /tmp/wheel_build/wheelhouse/* /var/nameko/wheelhouse; rm -rf /tmp/wheel_build

COPY config.yml /var/nameko/config.yml
COPY run.sh /var/nameko/run.sh
COPY alembic.ini /var/nameko/alembic.ini
ADD alembic /var/nameko/alembic

RUN chmod +x /var/nameko/run.sh

WORKDIR /var/nameko/

RUN . /appenv/bin/activate; \
    pip install --no-index -f wheelhouse nameko_examples_orders

EXPOSE 8000

CMD . /appenv/bin/activate; \
    /var/nameko/run.sh;

$ cat products/Dockerfile 
FROM debian:jessie

RUN apt-get update && \
    apt-get install -qyy \
    -o APT::Install-Recommends=false -o APT::Install-Suggests=false \
    python3 python-pip ca-certificates libpq-dev python-psycopg2 curl netcat rlwrap telnet  build-essential python3-dev && \
    cd /usr/local/bin && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

ADD . /tmp/wheel_build

RUN pip install virtualenv

RUN virtualenv -p python3 /appenv
RUN . /appenv/bin/activate; pip install -U pip; pip install wheel; cd /tmp/wheel_build; pip wheel -w /tmp/wheel_build/wheelhouse .; mkdir -p /var/nameko/wheelhouse; cp /tmp/wheel_build/wheelhouse/* /var/nameko/wheelhouse; rm -rf /tmp/wheel_build

COPY config.yml /var/nameko/config.yml
COPY run.sh /var/nameko/run.sh

RUN chmod +x /var/nameko/run.sh

WORKDIR /var/nameko/

RUN . /appenv/bin/activate; \
    pip install --no-index -f wheelhouse nameko_examples_products

EXPOSE 8000

CMD . /appenv/bin/activate; \
    /var/nameko/run.sh;

```

### Deploy with Gitlab.com and gitlab-runner @minikube

```
$ git init 
$ git add .
$ git commit -m "Init commit"
$ git remote add origin https://gitlab.com/nameko-microservices-examples-gitlab.git
$ git push origin master

$ kubectl create -f gitlab-runner-deployment.yaml


```

```
$ kubectl create clusterrolebinding default-sa-admin --user system:serviceaccount:default:default  --clusterrole cluster-admin

$ ./get-sa-token.sh --namespace default --account default

$ cat ca.crt | base64 > ca.crt.code


Gitlab: Setup CI/CD for project env variables:

CI_ENV_K8S_CA = cat ca.crt.code

CI_ENV_K8S_MASTER = https://192.168.99.100:8443

CI_ENV_K8S_SA_TOKEN = cat sa.token

```

```
Set up a specific Runner manually
Install GitLab Runner
Specify the following URL during the Runner setup: https://gitlab.com/ 
Use the following registration token during setup: -aRs3MMPqbP_HcxadMmg 
Reset runners registration token
Start the Runner!

$ kubectl get pod |grep runner
gitlab-runner-5d49c87d4f-d6rwj   1/1     Running   1          2d

kubectl exec -it gitlab-runner-5d49c87d4f-d6rwj /bin/bashroot@gitlab-runner-5d49c87d4f-d6rwj:/#gitlab-runner register --non-interactive --url "https://gitlab.com/" --registration-token "-aRs3MMPqbP_HcxadMmg" --executor "docker" --docker-image alpine:3 --description "docker-runner" --tag-list "docker-nameko-examples" --run-untagged --locked="false"

Runners activated for this project
 5c35da8d Pause Remove Runner
#567115
docker-runner

docker-nameko-examples
```

```
stages:
  - build
  - deploy

variables:

  CA: /etc/deploy/ca.crt


build:
  stage: build
  image: docker:latest
  services:
  - docker:dind
  
  script:
    - docker login --password=${DOCKER_PASSWORD} --username=${DOCKER_USERNAME}
    - apk add --no-cache make
    - TAG=latest make push-images

deploy_staging:
  tags:
  - docker-nameko-examples 
  stage: deploy
  image: davarski/k8s-helm:latest
  before_script:
    - apk add --no-cache make
    - mkdir -p /etc/deploy
    - echo ${CI_ENV_K8S_CA}|base64 -d > ${CA}
    - kubectl config set-cluster minikube --server=${CI_ENV_K8S_MASTER} --certificate-authority=/etc/deploy/ca.crt --embed-certs=true
    - kubectl config set-credentials default --token=${CI_ENV_K8S_SA_TOKEN}
    - kubectl config set-context minikube --cluster=minikube --user=default
    - kubectl config use-context minikube
    - helm init --client-only

  script:
    - helm install --name broker  --namespace examples stable/rabbitmq
    - helm install --name db --namespace examples stable/postgresql --set postgresDatabase=orders
    - helm install --name cache  --namespace examples stable/redis
    - cd k8s
    - NAMEPSACE=examples make deploy-namespace
    - NAMEPSACE=examples make install-gateway
    - NAMEPSACE=examples make install-orders
    - NAMEPSACE=examples make install-products 
  environment:
    name: staging
  only:
  - master
  ```
```Clean deployment:

 $ kubectl delete ns examples
namespace "examples" deleted
```
