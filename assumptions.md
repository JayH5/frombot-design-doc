# Assumptions
There are a few assumptions we need to make about how the Docker image build process is set up.

## Dockerfile repos
We assume that a version-controlled repository (i.e. a git repository) will be used that contains the Dockerfile(s) that the Docker image(s) is/are built from. In our case we use GitHub repos.

We're mostly concerned with repositories whose primary purpose is to manage Dockerfiles, not repositories that also contain the source code of the software actually being deployed in the Docker image.

Some examples of our Dockerfile repos:
* [`praekeltfoundation/docker-seed`](https://github.com/praekeltfoundation/docker-seed)
* [`praekeltfoundation/docker-django-bootstrap`](https://github.com/praekeltfoundation/docker-django-bootstrap)
* [`praekeltfoundation/docker-junebug`](https://github.com/praekeltfoundation/docker-junebug)

Ideally, the Dockerfile repo accurately describes everything about the Docker image that is produced from it. Building the Docker image from the repo can be reproduced reliably.

## Continuous Integration
We assume that Docker images will be built and pushed to a registry by some form of Continuous Integration (CI). In our case this is [Travis CI](https://travis-ci.org), but this system should apply to any _general-purpose_ CI service, such as [Semaphore](https://semaphoreci.com) or [CircleCI](https://circleci.com).

We are _not_ targeting images built using [Docker Hub's automated build system](https://docs.docker.com/docker-hub/builds/) as this is a very rudimentary CI system and already has its own mechanisms for chaining together automated builds.
