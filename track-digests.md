# How to track image digests
In order to track the exact image that an image is `FROM` in a Git repository, we need to store the image digest hash somehow. There are a few ways we could do this.

## Digest in the `FROM` tag
As described earlier in the [Docker tag section](docker-tags.md), it is possible to specify the image digest in the `FROM` tag:
```dockerfile
FROM python:2.7.13-slim@sha256:ca74abc54d21a6f1f16aaa483b931fe2f536eff5b9b5e77856c61173969605d2
```
or...
```dockerfile
FROM python@sha256:ca74abc54d21a6f1f16aaa483b931fe2f536eff5b9b5e77856c61173969605d2
```

The main problem with this is that the Docker builder will ignore the tag part of the image tag (`:2.7.13-slim`). It is possible to place any string there and as long as it is a syntactically valid tag, the parser will parse it, ignore it, and consider only the digest hash.

### Advantages:
* Docker will definitely build your image using exactly the same `FROM` image every time.
* Native Docker, not defining some new thing.

### Disadvantages:
* Tag information no longer checked. Could become incorrect.
* Adding the digest to tags is not commonly done.
* A bit clunky.

## Digest in a "metadata" comment
It could be possible just to add a comment to the Dockerfile with the image digest. This wouldn't be picked up by Docker itself, but a separate tool could. Most importantly, the Git repository would track the digest of the `FROM` tag.

Something like:
```dockerfile
# frombot: sha256:ca74abc54d21a6f1f16aaa483b931fe2f536eff5b9b5e77856c61173969605d2
FROM python:2.7.13-slim
```

### Advantages:
* Preserve tag information
* Fairly simple/lightweight
* All information contained in Dockerfile

### Disadvantages:
* Digest not enforced by Docker itself
* "Magic": Little explanation in Dockerfile about what this information is

## Digest in a label
Similar to the comment-based solution, this would add the `FROM` image digest to the Dockerfile. In this case we could store the information in a `LABEL` as metadata for the image:
```dockerfile
FROM python:2.7.13-slim
LABEL io.frombot.from-digest sha256:ca74abc54d21a6f1f16aaa483b931fe2f536eff5b9b5e77856c61173969605d2
```

### Advantages:
* Preserve tag information
* Still reasonably simple
* All information contained in Dockerfile
* Possible to inspect this label from a built image (for some purpose?)

### Disadvantages:
* Digest not enforced by Docker itself
* Still a bit magic

## Build arguments in the `FROM` tag
A [new feature](https://github.com/moby/moby/pull/31352) in Docker Engine 17.05.0 is support for [build arguments](https://docs.docker.com/engine/reference/builder/#arg) in the `FROM` tag. This means you could write a Dockerfile like this:
```Dockerfile
ARG FROM_TAG
FROM python:$FROM_TAG
RUN ...
```

And then build the image using a command like:
```
> $ docker build --tag myimage --build-arg FROM_TAG=sha256:adeb26c124912b3685cc69697d00ce1f71a3b6cb86b744dc75c324267b47caea .
```

If the build argument is not provided, though, things will break. A way to work around this could be to do something like this:
```Dockerfile
ARG FROM_DIGEST
FROM python:3.6${FROM_DIGEST:+@$FROM_DIGEST}
RUN ...
```

Doing `${FROM_DIGEST:+@$FROM_DIGEST}` means that the digest is only added if it has been set, and we add the `@` necessary for the syntax. If the digest is not specified, the existing tag, `3.6`, is used.

_Or_ we could even do _both_:
```Dockerfile
ARG FROM_TAG=3.6
ARG FROM_DIGEST
FROM python${FROM_TAG:+:$FROM_TAG}${FROM_DIGEST:+@$FROM_DIGEST}
RUN ...
```

### Advantages:
* Docker enforces the digest
* Can work with or without the build argument(s) being specified
* Preserve tag information

### Disadvantages:
* Does require changes to Dockerfiles
* Digest information not in Dockerfile itself
* Requires Docker 17.05.0+. As of writing there hasn't been a release in the "stable" channel with this functionality (first stable release with this functionality should be 17.07)
* The `ARG` defaulting is kind of verbose and ugly

## Digest in a "lockfile"
It could be possible to store digest hashes in a sort of makeshift lockfile, like those used by some package managers ([Bundler's](http://bundler.io) `Gemfile.lock` comes to mind).

Of course, such a lockfile would have to be enforced by a tool outside of Docker itself.

The lockfile could map Dockerfiles in a repo to digest hashes. An imaginary lockfile format with a YAML syntax called something like `frombot.lock` could look like:
```yaml
Dockerfile:
  tag: python:2.7.13
  digest: sha256:ca74abc54d21a6f1f16aaa483b931fe2f536eff5b9b5e77856c61173969605d2
```

A more complicated example with multiple Dockerfiles could look something like:
```yaml
debian/python/2.7/Dockerfile:
  tag: python:2.7.13-slim
  digest: sha256:ca74abc54d21a6f1f16aaa483b931fe2f536eff5b9b5e77856c61173969605d2

debian/python/3.6/Dockerfile:
  tag: python:3.6.1-slim
  digest: sha256:d70e4714326b99ddc9d4dc81d37b89e0becb6b46a01b2c0bb5f8d4029332a562

alpine/python/2.7/Dockerfile:
  tag: python:2.7.13-alpine
  digest: sha256:912be638dbb626dd1774b58a4afd0f279950fad57fd8cb13a57504dd998f9b00

alpine/python/3.6/Dockerfile:
  tag: python:3.6.1-alpine
  digest: sha256:6ebe18fd00f5175b5f1fe45bfb131f22f5d997f4fe361546cf0a13de396b8009
```

At this point, things start to look a lot more complicated and a bunch of questions arise. Do we then want a config file that tells our system where to look for the Dockerfiles and how to track tags? Is the design of this lockfile vaguely correct? Why am I adding the tags there? Should there be an address for the registry being used?

Anyway, before yak-shaving, let's _try_ to look at some of the pros/cons.

### Advantages:
* Purpose of the digest information is clear
* No changes to Dockerfiles
* Lots of flexibility for more functionality

### Disadvantages:
* Digest not enforced by Docker itself
* Complicated, lots of unanswered questions
