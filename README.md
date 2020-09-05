# multiarch-templates

OpenFaaS multi-arch templates

Notes:

This relies on Docker's new `buildx` container builder and buildkit. A flag is required to enable buildx: `export DOCKER_CLI_EXPERIMENTAL=enabled`, then commands can be invoked as per below

The basic workflow of the usual `faas-cli up` command invokes: `docker build`, `docker push` and the deploy code against the OpenFaaS gateway. These commands can also be run one-by-one.

When it comes to using buildx with faas-cli, there's currently no direct support, however you can still use a feature of faas-cli called shrinkwrap to help. The command `faas-cli build --shrinkwrap` will create a new folder with a valid Docker build context - first laying down the template of choice, followed by the function's code.

If you're looking for a HTTP-based template see: [OpenFaaS Golang HTTP templates](https://github.com/openfaas-incubator/golang-http-template)

Suggestions on how to support buildx and a series of platforms in [faas-cli](https://github.com/openfaas/faas-cli) would be welcomed. [Join us on Slack](https://slack.openfaas.io/) in the #contributors channel

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

## Node.js 12

As per above, but change the template name:


### Create a new function

```bash
faas-cli new --lang node12-multiarch $FN
```
