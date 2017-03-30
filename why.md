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

We need to find a way to trigger rebuilds of each image in the hierarchy (at least for the images we control).
