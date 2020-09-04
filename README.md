# multiarch-templates

OpenFaaS multi-arch templates

## Golang

### Create a new function

```bash
export DOCKER_CLI_EXPERIMENTAL=enabled
export OPENFAAS_PREFIX="alexellis2"
export FN="multiarch-fn"

faas-cli template pull https://github.com/alexellis/multiarch-templates
faas-cli new --lang go-multiarch $FN
```

### Build and push

```bash
docker buildx create --use --name=multiarch --node=multiarch
docker buildx build \
	--platform linux/amd64,linux/arm/v7,linux/arm64 \
	--output "type=image,push=true" \
	--tag $OPENFAAS_PREFIX/$FN:latest build/$FN/
```

### Just build

```bash
docker buildx create --use --name=multiarch --node=multiarch
docker buildx build \
--output "type=docker,push=false" \
--tag $OPENFAAS_PREFIX/$FN:latest build/$FN/
```

### Deploy your function

```bash
faas-cli deploy --image $OPENFAAS_PREFIX/$FN:latest $FN
```