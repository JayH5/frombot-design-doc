# What already exists
## MicroBadger
[MicroBadger](https://microbadger.com) provides a few tools for tracking information about Docker images. This mostly involves badges for READMEs and webhook-based notifications of image changes.

### Tying source code to a Docker image
One of the big things MicroBadger tries to do is tie source code in a Git repository with Docker images in a registry (only Docker Hub is supported currently). They explain this better than I probably can on their [About page](https://microbadger.com/about).

The way they're tying Git repos to Docker images is interesting. They require Dockerfile authors to use the new container [Label Schema Convention](http://label-schema.org) to add metadata to their images that points to the Git repo that defines the Dockerfile. For example, the following lines could be added to a Dockerfile:
```dockerfile
ARG VCS_REF
LABEL org.label-schema.vcs-ref=$VCS_REF \
      org.label-schema.vcs-url="https://github.com/praekeltfoundation/docker-vumi"
```

This may be useful at some point but complicates the build process and there are questions around layer caching that I don't know the answers to yet. And once you've added that metadata all you get is a link to the Git repo on the MicroBadger page and you can add a badge to your repo with the value you provided for the `org.label-schema.vcs-ref` label.

### Image layer analysis
Even without the VCS metadata, though, MicroBadger provides some interesting information about the layers in an image. For example, see the [MicroBadger page for `praekeltfoundation/django-bootstrap`](https://microbadger.com/images/praekeltfoundation/django-bootstrap).

MicroBadger is able to detect the parent images of the image. It provides information on the on-disk size of each image layer. Perhaps most interestingly, it can group tags together that point to the same image. For example, it can tell that the `2.7.13-slim`, `2.7-slim`, and `2-slim` tags for the `python` image all point to the same thing.

### Notifications
Some early functionality in MicroBadger is to provide notifications when changes occur to images. This is done via webhooks. You give MicroBadger an image name and a webhook URL and it will call the webhook when that image is updated. This webhook could be used to trigger a build of your image.

This system _could_ be used to implement the system we're talking about but there are a few limitations:
* It is quite a manual process setting up these notifications in the MicroBadger UI.
* _You can set up notifications for one base image for free. If you'd like additional notifications please apply for our private beta._
* Notifications won't update the Git repo for your Dockerfile.

## Docker Hub automated builds
As mentioned in the [assumptions section](assumptions.md), we're not looking at using Docker Hub automated builds. Still, there are some tools here for automating image builds which are of interest.

### Repository links
[Repository links](https://docs.docker.com/docker-hub/builds/#repository-links) make it possible to _link_ one or more Docker images in Docker Hub to your image. When one of the linked images is updated, a new build of your image is triggered. This is pretty much what we want for our system but has a few issues:
* No notification of when these new builds happen (without manually setting up a webhook).
* No updates to the Git repo that defines the Docker image.
* All the other issues with Docker Hub automated builds, as outlined in the [assumptions](assumptions.md) document.

### Build triggers
[Remote Build Triggers](https://docs.docker.com/docker-hub/builds/#remote-build-triggers) are basically webhooks you can use to trigger the Docker Hub build. You send a `POST` request with a bit of JSON that can specify things like Docker tag or Git branch or tag to build.

This is perhaps what the MicroBadger people had in mind when designing their notification system.

### Webhooks
It's possible to have Docker Hub call a [webhook](https://docs.docker.com/docker-hub/webhooks/) when an event around an automated build occurs. This may be useful for triggering a build somewhere outside Docker Hub or a redeploy of an updated image.

## Other solutions
[Here](http://stackoverflow.com/questions/26423515/how-to-automatically-update-your-docker-containers-if-base-images-are-updated) is a StackOverflow discussion covering roughly the problem we're trying to solve, although most people seem to be more focussed on the problem of ensuring a _running_ container is using the latest version of its image. For now, we're just concerned with keeping images up-to-date, not containers.

The solutions people propose include:
* Bash scripts that periodically pull the latest image (don't seem to be solutions to actually _building_ images)
* Running `unattended-upgrade` in a cron job in the containers so that they stay up-to-date (_not_ something we want to do)
* Using Docker Hub's automated build features
* _Such a tool would be good to have, but there's nothing that I know of that will do this._
* Docker Hub webhooks with IFTTT
* MicroBadger (suggested by a MicroBadger developer)

Ultimately, none of these solutions really solve what we're trying to solve here.

## Docker official images
What does the team that maintains the collection of official images do about this problem? They don't do anything :-/

To be fair, the problem is slightly lessened with the official images as the depth of the image hierarchy is generally quite short-- most of the official images are one or two steps away from an empty image. These are really the images we want to _watch_ for updates.

Still, it may be worth looking at the build system that has been developed. This system is centred around the [`official-images`](https://github.com/docker-library/official-images) repository. In this repository there are a set of ["library definition files"](https://github.com/docker-library/official-images#library-definition-files). These files contain instructions for building images from a Git repository using a tool called [Bashbrew](https://github.com/docker-library/official-images/tree/master/bashbrew). The library definition files list the Git repository address, commit hash (or other reference), and directory in that repository for a Dockerfile. These references are mapped to Docker tags that should be built from the Dockerfiles they describe. This means that the _Dockerfile_ used to build each image is always consistent.

Generally, some script will be used to generate these library definition files. For example, in the `docker-library/python` repository that defines the official Python images, [here](https://github.com/docker-library/python/blob/master/generate-stackbrew-library.sh).

Unfortunately, this doesn't solve the problem of `FROM` tags pointing to moving targets. So the resulting image is not guaranteed to be the same between builds (or even guaranteed to build). Images will only be updated when they are rebuilt. So, for example, the Python images will only receive an updated Debian (or Alpine Linux) base image when they are rebuilt for some reason (unless somebody manually rebuilds them for this purpose).
