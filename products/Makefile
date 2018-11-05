TAG ?= dev
# docker

build-image:
	docker build -t davarski/nameko-example-products-circleci:$(TAG) .;

push-image: build-image
	docker push davarski/nameko-example-products-circleci:$(TAG)
