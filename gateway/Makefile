TAG ?= dev

# docker


build-image:
	docker build -t davarski/nameko-example-gateway-circleci:$(TAG) .;

push-image: build-image
	docker push davarski/nameko-example-gateway-circleci:$(TAG)
