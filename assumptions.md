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
### Using a general-purpose CI service
We assume that Docker images will be built and pushed to a registry by some form of Continuous Integration (CI). In our case this is [Travis CI](https://travis-ci.org), but this system should apply to any _general-purpose_ CI service, such as [Semaphore](https://semaphoreci.com) or [CircleCI](https://circleci.com).

We want to use an existing, full-featured CI service because:
* We have a lot of existing experience with Travis CI (and we already pay them for their service).
* We want full control over the build environment, particularly over which version of Docker we use.
* It's easier to run integration tests against built images.
* We've built some existing tooling to make tasks like tagging images easier inside Travis CI ([docker-ci-deploy](https://github.com/praekeltfoundation/docker-ci-deploy)).

### Why not Docker Hub automated builds?
[Docker Hub's automated builds](https://docs.docker.com/docker-hub/builds/) are a quick and easy way to get started with automating building Docker images. It also provides some tools to handle the problem we're seeking to solve by making it possible to trigger builds of images when another image is updated. See the [document about existing solutions](already-exists.md) for more information.

So why not use this? There are a few reasons, the primary one being that Docker Hub automated builds don't provide enough control over the build process. Most of the advantages of a full-featured CI service listed above do not apply to Docker Hub automated builds.

Other problems we've encountered:
* No notifications of failed builds (unless you manually set up a webhook).
* Enjoy refreshing the page as you watch the build logs.
* Very limited control of image tagging.
* Obscure [build hook](https://docs.docker.com/docker-cloud/builds/advanced/#build-hook-examples) system for more advanced usage basically overrides anything you set in the Docker Hub web UI.

In addition to this, in the past we've experienced poor reliability from Docker Hub automated builds, and the Docker organisation generally provides little support if you are not a paying customer. Documentation is lacking, to say the least.
