# multiarch-templates

OpenFaaS multi-arch templates

Notes:

* The OpenFaaS CLI doesn't have a feature to invoke "buildx", Docker's next-gen container builder, so commands have to be run one by one.
* An experimental flag is required for buildx`export DOCKER_CLI_EXPERIMENTAL=enabled`
* `faas-cli build --shrinkwrap`, it generates a build context from the function's template and code.

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
faas-cli build --shrinkwrap -f $FN.yml

docker buildx create --use --name=multiarch --node=multiarch
docker buildx build \
	--platform linux/amd64,linux/arm/v7,linux/arm64 \
	--output "type=image,push=true" \
	--tag $OPENFAAS_PREFIX/$FN:latest build/$FN/
```

### Just build

```bash
faas-cli build --shrinkwrap -f $FN.yml

docker buildx create --use --name=multiarch --node=multiarch
docker buildx build \
--output "type=docker,push=false" \
--tag $OPENFAAS_PREFIX/$FN:latest build/$FN/
```

### Deploy your function

The build above will work on amd64, RPi (armv7) and ARM64.

```bash
faas-cli deploy --image $OPENFAAS_PREFIX/$FN:latest \
  --name $FN

curl http://127.0.0.1:8080/function/multiarch-fn -d "hi"
Hello, Go. You said: hi
```
