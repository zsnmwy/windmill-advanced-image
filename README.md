# Windmill Advanced Image

## Introduction

Automatically build images base on windmill images.

Make up for the shortcomings of the official image.

Only trigger the build when the windmill release exactly version. `1.110`, `1.117` etc.

## How

- Check the official image version By [Github Actions - check.yml](https://github.com/zsnmwy/windmill-advanced-image/blob/master/.github/workflows/check.yml) Every hours.

- If the official image version is updated, the latest image information `windmill-tags.json` will commit to master branch.
    > `skopeo list-tags docker://ghcr.io/windmill-labs/windmill | tee windmill-tags.json`

- [The Github Actions - build.yaml](https://github.com/zsnmwy/windmill-advanced-image/blob/master/.github/workflows/build.yaml) will be triggered again, and the image will be generated and pushed to the [GHCR](https://github.com/zsnmwy/windmill-advanced-image/pkgs/container/windmill).

It will be scan the Docker folders. And use the matrix feature of the Github Action to build the image.

If the folders has these files.
```
.
├── Docker
│   ├── Helm
│   └── OCR
```

Will be get these images. 

- Both build the CE and EE version. `ce` and `ee`.
- The dockerfile name will be translate to the image tag name. Upper case to lower case. `Helm` -> `helm`, `OCR` -> `ocr`.
- The image tag will be add the windmill version. `ce-helm-1.117`, `ee-helm-1.117`, `ce-ocr-1.117`, `ee-ocr-1.117`.

```bash
# The windmill version is 1.117.

ghcr.io/zsnmwy/windmill:ce-helm-1.117
ghcr.io/zsnmwy/windmill:ee-helm-1.117
ghcr.io/zsnmwy/windmill:ce-ocr-1.117
ghcr.io/zsnmwy/windmill:ee-ocr-1.117
```

## Usage

- Pull the image from the GHCR if the image meets your expect.

```bash
docker pull ghcr.io/zsnmwy/windmill:ce-helm-1.117
```

- Fork this repo and add your own Dockerfile.

- Contributing your Dockerfile to this repo.

## Contributing

The Dockerfile is must be accept the `--build-arg WINDMILL_VERSION=xxxx` to build the image.

Example:

```dockerfile
ARG WINDMILL_VERSION=ghcr.io/windmill-labs/windmill:main

FROM $WINDMILL_VERSION
```

The CI will be pass the `WINDMILL_VERSION` to the Dockerfile.

```yaml
- name: Build and push
    uses: docker/build-push-action@v4
    with:
    context: .
    platforms: linux/amd64
    push: true
    file: Docker/${{matrix.docker_files}}
    # here
    build-args: |
        WINDMILL_VERSION=ghcr.io/windmill-labs/windmill-ee:${{env.windmill_version}}
    tags: |
        ghcr.io/${{ github.repository_owner }}/windmill:ee-${{env.image_name}}-${{env.windmill_version}}
```