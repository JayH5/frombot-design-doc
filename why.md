# Why it's important to track `FROM` images
Take, for example, our Junebug image. The image hierarchy looks something like this:
```
         debian:jessie
               |
               v
      python:2.7.13-slim
               |
               v
praekeltfoundation/python-base:2
               |
               v
    praekeltfoundation/vumi
               |
               v
  praekeltfoundation/junebug
```

The `python` image's Dockerfile starts with `FROM debian:jessie` and the `python-base` with `FROM python:2.7.13-slim`, and so on. Thus, a hierarchy of images is created.

A problem arises when a lower image is updated. If one of the images _before_ the Junebug image is updated, every image up to and including the Junebug image must be rebuilt.

Without automation, changes are slowly propagated through the chain, usually without us knowing. For example:
* 2016/11/04: The `debian:jessie` tag is updated [here](https://github.com/docker-library/official-images/commit/7d72868b7edd97b40c8092e15a017573452c99e8) for [CVE-2016-6321](https://security-tracker.debian.org/tracker/CVE-2016-6321) which describes a vulnerability in `tar`, a very commonly-used tool.
* 2016/11/17: The `python:2.7.12-slim` tag (we hadn't got to `2.7.13` yet) is eventually updated as part of a batch of updates [here](https://github.com/docker-library/official-images/commit/1dd05cdd4f4ad2b4671746c82ea77d0866306a13#diff-051bb6df61b6c6e7f8c1868985011b07). This also included a newer version of `pip`.
* 2016/11/21: The `praekeltfoundation/python-base:2` tag is updated after [this PR](https://github.com/praekeltfoundation/dockerfiles/pull/42) is merged, triggering a new build.
* 2017/01/10: The `praekeltfoundation/vumi` tag is finally updated when a new version of Vumi is released and updated in [this PR](https://github.com/praekeltfoundation/docker-vumi/pull/14).
* 2017/01/10: The `praekeltfoundation/junebug` tag is updated when a new version of Junebug is released and updated in [this PR](https://github.com/praekeltfoundation/docker-junebug/pull/19).

So just over 2 months for a security fix to propagate through all of the images. This is, in a way, exaggerated, as by the time the Vumi image was rebuilt there were even more security fixes and a minor Python release (2.7.13) that had been built into the `python-base` image.

We need to find a way to trigger rebuilds of each image in the hierarchy, at least for the images we control, in an automated fashion.
