TAG ?= dev

# docker


build-image:
	docker build -t davarski/nameko-example-gateway-circleci:$(TAG) -f docker.run .;

push-image:
	docker push davarski/nameko-example-gateway-circleci:$(TAG)
