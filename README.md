# Docker studies :whale2::pencil:

## Images size :anchor:

Suppose we need an image with `Python version 3.10.5` and `Poetry version 1.1.14`.

We can use a Python image fully loaded (with `git`, `curl`, etc) and install Poetry in it. That is what is done in [`docker/size/Dockerfile.single_stage`](docker/size/Dockerfile.single_stage).

Or we can use a Python image fully loaded only to download the installation script for poetry, but install it in a Python small image. That is what is done in [`docker/size/Dockerfile.multi_stage`](docker/size/Dockerfile.multi_stage).

To build the images described above, run
```
docker build --tag size-single-stage --file docker/size/Dockerfile.single_stage ./
docker build --tag size-multi-stage --file docker/size/Dockerfile.multi_stage ./
```

Let's check the size of the images created
```
$ docker image ls -a
REPOSITORY                  TAG                    IMAGE ID       CREATED          SIZE
size-multi-stage            latest                 57bbdaaa976a   48 seconds ago   199MB
size-single-stage           latest                 eca596905b85   2 minutes ago    994MB
```

The image using the strategy of **Multi Stage** build is about 5 times smaller then the image that uses Single Stage build.

## References :books:

- https://testdriven.io/blog/docker-best-practices/
- https://github.com/badtuxx/DescomplicandoDocker
- https://www.youtube.com/playlist?list=PLf-O3X2-mxDn1VpyU2q3fuI6YYeIWp5rR
