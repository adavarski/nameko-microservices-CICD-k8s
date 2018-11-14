image: docker:latest

services:
  - docker:dind

stages:
- build

build:
  stage: build
  script:
    - docker login --password=${DOCKER_PASSWORD} --username=${DOCKER_USERNAME}
    - apk add --no-cache make
    - TAG=latest make push-images

