# CircleCI Images [![CircleCI Build Status](https://circleci.com/gh/circleci/circleci-images.svg?style=shield)](https://circleci.com/gh/circleci/circleci-images) [![GitHub license](https://img.shields.io/badge/license-MIT-blue.svg)](https://raw.githubusercontent.com/circleci/circleci-docs/master/LICENSE) [![CircleCI Community](https://img.shields.io/badge/community-CircleCI%20Discuss-343434.svg)](https://discuss.circleci.com)

```
Example:
  image: redis@sha256:54057dd7e125ca41afe526a877e8bd35ec2cdd33b9217e022ed37bdcf7d09673
```

A set of convenience images that work better in context of CI.  This repo contains the official set of images that CircleCI maintains.  It contains language as well as services images:

* Language images (e.g. `ruby`, `python`, `node`) are images targeted for common programming languages with the common tools pre-installed.  They primarily extend the [official images](#official-images) and install additional tools (e.g. browsers) that we find very useful in context of CI.
* Service images (e.g. `mongo`, `postgres`) are images that have the services pre-configured with development/CI mode.  They also primarily extend the corresponding [official images](#official-images) but with sensible development/CI defaults (e.g. disable production checks, default to nojournal to speed up tests)

## Official Images

We extend [Docker Official Repositories](https://docs.docker.com/docker-hub/official_repos/) in order to start with the same consistent set of images.

This allows us to make things more standardized. From our scripts for checking for updates, the type of OS on the base image, and so forth. We can recommend using `apt-get install` rather than documenting various constraints depending on which stack you're using.

The official images on Docker Hub are curated by Docker as their way to provide convenience images which address both development and production needs. Since Docker sponsors a dedicated team responsible for reviewing each of the official images, we can take advantage of the community maintaining them independently without trying to track all of the sources and building automations for each one. For now we can take a shortcut, without building this infrastructure.

Finally, our convenience images are augmenting these official images, by adding some missing packages, that we install ourselves for common dependencies shared for the CI environment.

All of the official images on Docker Hub have an "_" for the username, for example:
https://hub.docker.com/_/ruby

You can view all of the officially supported images here:
https://hub.docker.com/explore/

CircleCI supported images are here:
https://hub.docker.com/r/circleci/

To view the Dockerfiles for CircleCI images, visit the [CircleCI-Public/circleci-dockerfiles](https://github.com/circleci-public/circleci-dockerfiles) repository.

# How to add a bundle with images

A bundle is a top-level subfolder in this repository (e.g. `postgres`).

For the image Dockerfiles, we use a WIP templating mechanism.  Each bundle should contain a `generate-images` script for generating the Dockerfiles.  You can use [`postgres/generate-images`](postgres/generate-images) and [`node/generate-images`](node/generate-images) for inspiration.  The pattern is executable script of the following sample:


```bash
#!/bin/bash

# the base image we should be tracking.  It must be a Dockerhub official repo
BASE_REPO=node

# Specify the variants we need to publish.  Language stacks should have a
# `browsers` variant to have an image with firefox/chrome pre-installed
VARIANTS=(browsers)

# By default, we don't build the alpine images, since they are typically not dev friendly
# and makes our experience inconsistent.
# However, it's reasonable for services to include the alpine image (e.g. psql)
#
# uncomment for services

#INCLUDE_ALPINE=true

# if the image needs some basic customizations, you can embed the Dockerfile
# customizations by setting $IMAGE_CUSTOMIZATIONS.  Like the following
#

IMAGE_CUSTOMIZATIONS='
RUN apt-get update && apt-get install -y node
'

# boilerplate
source ../shared/images/generate.sh
```

By default, the script uses `./shared/images/Dockerfile-basic.template` template which is most appropriate for language based images.  Language image variants (e.g. `-browsers` images that have language images with browsers installed) use the `./shared/images/Dockerfile-${variant}.template`.

Service image should have their own template.  The template can be kept in `<bundle-name>/resources/Dockerfile-basic.template` - like [`./mongo/resources/Dockerfile-basic.template`](./mongo/resources/Dockerfile-basic.template).

To build all images - push a commit with `[build-images]` text appearing in the commit message.

Also, add the bundle name to in Makefile `BUNDLES` field.


## Development

*This section is a work in progress*

This project's build system is managed with `Make`.
It generates Dockerfiles and builds images based off of upstream Docker Library base images, variant images that upstream may have, variant images that CircleCI provides, as well as tools common for testing projects in various programming languages.

### Generate Dockerfiles

Use `make <base-image>/generate_images` to generate of all of the Dockerfiles for a specific base image.
For example, to generate all of the Dockerfils for the `circleci/golang` image, you would run `make golang/generate_images`.

Use `make images` to generate Dockerfiles for **every** base image we have.
This process is fairly fast with decent Internet connection.

Once generated, Dockerfiles for each image will be in the images folder for that base image (e.g. ./python/images/).

### Build Images

#### Build a single Dockerfile

You can build a single image, with a throway name and the `docker build` command.
`docker build -t <throw-away-img-name> <directory-containing-dockerfile>` 
Here's how you would build the "regular" Go v1.11 CircleCI image:

`docker build -t test/golang:latest golang/images/1.11.0/`

The image name and tag (`test/golang:latest`) can be whatever name you want, it's just for local dev and can be deleted.

#### Build all Dockerfiles for a base image

Use `make <base-image>/publish_images` to `docker build` all of the Dockerfiles for that base image.
There's a couple things to note here:

1. Each base image has **a lot** of images.
Building them will end up taking several GBs of disk space, and can take quit a while to run.
Make sure you want to do this before you do it.
1. The build script will also try to run `docker push`.
If you don't work for CircleCI, this will fail and that's okay.
It's safe to ignore.

#### Build all Dockerfiles for every base image

Don't do it.
If you have an Ultrabook laptop, it won't be happy.
If you really want to do it, look at the `Makefile`.


## Limitations
* The template language is WIP - it only supports `{{BASE_IMAGE}}` template.  We should extend this.
* Generated Dockerfiles isn't checked into repo.  Since we track moving set of tags, checking into repository can create lots of unnecessary changes.
* By default, the `staging` branch of this repository pushes to the [`ccistaging` Docker Hub org](https://hub.docker.com/r/ccistaging).  Once we get some test builds with these images, we can promote them to the [`circleci` Docker Hub org](https://hub.docker.com/r/circleci) by merging changes from the `staging` branch into the `master` branch.
* We cannot support Oracle JDK for licensing reasons. See [Oracle's Binary Code License Agreement for the Java SE Platform](http://oracle.com/technetwork/java/javase/terms/license/index.html) and [Stack Exchange: Is there no Oracle JDK for docker?](https://devops.stackexchange.com/questions/433/is-there-no-oracle-jdk-for-docker) for details.

## Licensing
The `circleci-images` repository is licensed under The MIT License. See [LICENSE](https://github.com/moby/moby/blob/master/LICENSE) for the full license text.
