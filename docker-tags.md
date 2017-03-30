# About Docker tags
Docker image tags have the form:
```
[REGISTRYHOST/]NAME[:TAG][@DIGEST]
```
Where:
* `REGISTRYHOST` is the address to the registry (usually omitted for images stored in the Docker Hub registry), e.g. `qa-mesos-persistence.za.prk-host.net:5000`
* `NAME` is the name of the image. This is the only required part of the image tag, e.g. `praekeltfoundation/vumi`
* `TAG` is the tag information for the image, e.g. `:latest` or `0.9.11`
* `DIGEST` is a sha256 hash of the image digest

See the [full reference](https://github.com/docker/distribution/blob/v2.6.0/reference/reference.go) for more details about valid values for each of these fields.

Docker tags are kind of like git tags-- they are just _pointers_ to a certain thing (in this case an image) and are _mutable_ and can change over time.

In git-land we usually don't move tags about, but in Docker-land it's pretty common. A tag like `python:2.7.13-slim` may seem quite specific but the image it points to could change when it is rebuilt for any number of reasons:
* The `debian:jessie` image it is built on receives security updates and changes
* There is some change to the way Python is built
* A new version of `pip` is released
* Any other changes to [the Dockerfile](https://github.com/docker-library/python/blob/master/2.7/slim/Dockerfile)

We need some way to trigger a rebuild of an image if the image its `FROM` tag points to changes.

## Why not use _immutable_ tags?
Some people do use immutable tags. For example, it would be possible to tag images with the git sha that the image was created from:
```
praekeltfoundation/vumi:06fe87c79bc0ea8ada5f8048dac5b4a6895314ea
```

This would work for our images but has a couple of problems:
 1. It's not very user-friendly. We lose the Vumi version information.
 2. It doesn't work for images that _aren't_ tagged this way (e.g. the `python` image).

Another way to do things might be to always use the full digest in the `FROM` declaration:
```dockerfile
FROM python:2.7.13-slim@sha256:ca74abc54d21a6f1f16aaa483b931fe2f536eff5b9b5e77856c61173969605d2
```
or...
```dockerfile
FROM python@sha256:ca74abc54d21a6f1f16aaa483b931fe2f536eff5b9b5e77856c61173969605d2
```

In the first case, the `:2.7.13-slim` tag part is actually ignored by Docker and it just uses the digest hash-- so the tag is fairly meaningless. :-/

Whether the Docker tags are mutable or immutable, the repo that defines the Dockerfile still needs to be updated in some way so as to trigger a new build. This should be automated.
