TAG ?= dev
# docker

build-image:
	docker build -t davarski/nameko-example-orders-circleci:$(TAG) .;

push-image:
	docker push davarski/nameko-example-orders-circleci:$(TAG)
