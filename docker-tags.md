# About Docker tags
## Tag format
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

## Tags are mutable
Docker tags are kind of like git tags-- they are just _pointers_ to a certain thing (in this case an image) and are _mutable_ and can change over time.

In git-land we usually don't move tags about, but in Docker-land it's pretty common. A tag like `python:2.7.13-slim` may seem quite specific but the image it points to could change when it is rebuilt for any number of reasons:
* The `debian:jessie` image it is built on receives security updates and changes
* There is some change to the way Python is built
* A new version of `pip` is released
* Any other changes to [the Dockerfile](https://github.com/docker-library/python/blob/master/2.7/slim/Dockerfile)

We need some way to trigger a rebuild of an image if the image its `FROM` tag points to changes.

### Why not use tags that _don't_ change?
Some people do do this. For example, it would be possible to tag images with the git sha that the image was created from:
```dockerfile
FROM praekeltfoundation/vumi:06fe87c79bc0ea8ada5f8048dac5b4a6895314ea
```
or perhaps...
```dockerfile
FROM praekeltfoundation/vumi:0.6.13-06fe87c
```
or even doing something like Debian packaging and adding the build number...
```dockerfile
FROM praekeltfoundation/vumi:0.6.13-2
```

This would work for our images but has a few problems:
 1. It's not very user-friendly. Where does that hash come from? What does it mean? The tag then includes version information for both the source software and the Dockerfile. Which is which?
 2. In the third case, keeping track of a (useful) build number in an external CI service can be tricky.
 2. It doesn't work for images that _aren't_ tagged this way (e.g. the `python` image).

> Note that I'm not saying that these are ways of tagging an image that should **never** be used-- in fact I think they have their places. But they don't solve the underlying problem we have here.

Another way to do things might be to always use the image digest:
```dockerfile
FROM python:2.7.13-slim@sha256:ca74abc54d21a6f1f16aaa483b931fe2f536eff5b9b5e77856c61173969605d2
```
or...
```dockerfile
FROM python@sha256:ca74abc54d21a6f1f16aaa483b931fe2f536eff5b9b5e77856c61173969605d2
```

In the first case, the `:2.7.13-slim` tag part is actually ignored by Docker and it just uses the digest hash-- so the tag is fairly meaningless. :-/

Whether the Docker tags are moving targets or not, the repo that defines the Dockerfile still needs to be updated in some way so as to trigger a new build and doing this manually is a pain.

## Semantic version tags
A convention seems to have been adopted by the maintainers of the [Docker official images](https://github.com/docker-library/official-images/) for working with semantic version information in image tags. It works something like this:
```
x.y.z[-VARIANT]
```
where:
* `x`, `y`, `z` are the major, minor, and patch version numbers.
* `VARIANT` describes some special variation of the image, e.g. `slim` or `onbuild`

The image is also tagged with "less precise" version information, if it is the latest version in the series:
```
x.y[-VARIANT]    # if x.y.z is the latest x.y version
x[-VARIANT]      # if x.y is the latest x version
```

Additionally, the image is either tagged with the variant or the tag `latest` if it is the default variant:
```
{latest|VARIANT} # if x is the latest version overall
```

For an example of this, see the tags for [Python official image](https://hub.docker.com/r/library/python/).

Our [docker-ci-deploy](https://github.com/praekeltfoundation/docker-ci-deploy) tool uses this versioning convention when the `--tag-semver` option is used.
