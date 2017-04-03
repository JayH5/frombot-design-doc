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
