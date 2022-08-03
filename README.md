# Docker studies :whale2::pencil:

Studies about how to best use Docker :rocket:

## Images size :anchor:

To make your images smaller, check this section.

### Multi-stage build :baggage_claim:
:link: [Docker oficial documentation](https://docs.docker.com/develop/develop-images/multistage-build/)

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

The image using the strategy of **Multi-stage build** is about 5 times smaller then the image that uses Single Stage build.

### Layers :paw_prints:
:link: [Docker oficial documentation](https://docs.docker.com/storage/storagedriver/)

When we build an image, each `RUN`, `COPY` and `ADD` instruction in the Dockerfile creates a **layer**.

After a **layer** concludes, it becomes read only. This means we can not modify it.

We can check the **layers** of one of the previous images below.
```
$ docker history size-multi-stage
IMAGE          CREATED              CREATED BY                                      SIZE      COMMENT
9728fad198bb   About a minute ago   /bin/sh -c python ./install_poetry.py     &&…   74.2MB
2a51b9b3ae9d   About a minute ago   /bin/sh -c #(nop) COPY file:86f8cfaaafaefdbc…   27.3kB
70b198e593fe   About a minute ago   /bin/sh -c #(nop)  ENV PYTHONDONTWRITEBYTECO…   0B
014ac54db0a0   5 days ago           /bin/sh -c #(nop)  CMD ["python3"]              0B
<missing>      5 days ago           /bin/sh -c set -eux;   savedAptMark="$(apt-m…   11.4MB
<missing>      5 days ago           /bin/sh -c #(nop)  ENV PYTHON_GET_PIP_SHA256…   0B
<missing>      5 days ago           /bin/sh -c #(nop)  ENV PYTHON_GET_PIP_URL=ht…   0B
<missing>      2 weeks ago          /bin/sh -c #(nop)  ENV PYTHON_SETUPTOOLS_VER…   0B
<missing>      2 weeks ago          /bin/sh -c #(nop)  ENV PYTHON_PIP_VERSION=22…   0B
<missing>      2 weeks ago          /bin/sh -c set -eux;  for src in idle3 pydoc…   32B
<missing>      2 weeks ago          /bin/sh -c set -eux;   savedAptMark="$(apt-m…   30.1MB
<missing>      2 weeks ago          /bin/sh -c #(nop)  ENV PYTHON_VERSION=3.10.5    0B
<missing>      2 weeks ago          /bin/sh -c #(nop)  ENV GPG_KEY=A035C8C19219B…   0B
<missing>      2 weeks ago          /bin/sh -c set -eux;  apt-get update;  apt-g…   3.11MB
<missing>      2 weeks ago          /bin/sh -c #(nop)  ENV LANG=C.UTF-8             0B
<missing>      2 weeks ago          /bin/sh -c #(nop)  ENV PATH=/usr/local/bin:/…   0B
<missing>      2 weeks ago          /bin/sh -c #(nop)  CMD ["bash"]                 0B
<missing>      2 weeks ago          /bin/sh -c #(nop) ADD file:d978f6d3025a06f51…   80.4MB
```

The `COPY` instruction added 27.3kB to our image, and if we delete the file in a next **layer**, it will still have that size in the final image, because that **layer** (and all of them, except the one the Container is running, like the `ENTRYPOINT`, that is read and write) is read only.

If we sum up the sizes of the `<missing>` **layers** (which come from the base image `3.10.5-slim-bullseye`), we can see it is equal to the image size.
```
$ docker image ls "*slim*"
REPOSITORY                  TAG                    IMAGE ID       CREATED         SIZE
python                      3.10.5-slim-bullseye   014ac54db0a0   5 days ago      125MB
```

Because of this, is important to do have a `RUN` instruction like this
```
RUN apt-get update && apt-get install --yes \
  package1 \
  package2 \
  ...
  packageN \
  && rm -rf /var/lib/apt/lists/*
```
in your Dockerfile, that removes cache before it becoming read only.

## Performance :running:

To make Docker faster, check this section.

### Build context :earth_americas:

:link: [Docker oficial documentation](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/#understand-build-context)

**Build context** is the required parameter for `docker build` command (and present in Docker compose as well in the `build` section, as the `context` parameter).

**Build context** can increase the time to build an image. Check the output of the following commands.
```
$ docker build --tag big-context --file docker/size/Dockerfile.single_stage ./
Sending build context to Docker daemon  48.64kB
...
```

```
$ docker build --tag medium-context --file docker/size/Dockerfile.single_stage ./docker/size
Sending build context to Docker daemon  3.584kB
...
```

```
$ docker build --tag small-context - < docker/size/Dockerfile.single_stage
Sending build context to Docker daemon  2.048kB
...
```

If you do not need files in the root of a project, send a different **Build context** to Docker to speed it up, or use a `.dockerignore` file.

## References :books:

- https://testdriven.io/blog/docker-best-practices/
- https://github.com/badtuxx/DescomplicandoDocker
- https://www.youtube.com/playlist?list=PLf-O3X2-mxDn1VpyU2q3fuI6YYeIWp5rR
- https://docs.docker.com/develop/develop-images/dockerfile_best-practices/
