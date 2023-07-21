---
title: The docker:// shorthand and stacker Use.
tags: ubuntu
published: false
---
# The docker:// shorthand and stacker Use.
There are lots of "just works" behavior and ease of use features in docker.

One example is the 'docker://ubuntu:latest' or 'docker://busybox'.
These are shortcuts for 'docker://docker.io/library/ubuntu:latest' and 'docker://library/busybox' respectively.

It took me a long time to fully understand this.

Effectively, the default 'host' portion of a 'docker://' url is "docker.io/library/".  What if you typed 'wget https://my.txt' and that expanded to 'https://netscape.net/my.txt'?  That would seem weird, right?  It is weird here too.

I wasn't able to get the right google query to even find documentation of this behavior.  [This stackoverflow post](https://stackoverflow.com/questions/66393937/naming-to-docker-io-explanation-and-prevention) was the closest thing I could find.


## Stacker substitution of docker://
Finally groking this gave me insight to make stacker files less dependent on this specific network resource. For example see the stacker.yaml below:

```yaml
mylayer:
  build_only: true
  from:
    type: "docker"
    url: ${{DOCKER_LIBRARY:docker://docker.io/library/}}busybox:latest
  run: |
    busybox --list | head -n 10
    exit 1 ; # just to force a re-build
```

To build with the default value of DOCKER_LIBARY, just run `stacker build`.

If you want to work offline, you can do:

```
$ skopeo copy docker://busybox:latest oci:/tmp/my.oci.d:busybox:latest
$ stacker build --substitute=DOCKER_LIBRARY=oci:/tmp/my.oci.d:
preparing image mylayer...
loading oci:/tmp/my.oci.d:busybox:latest
Copying blob 3f4d90098f5b skipped: already exists  
Copying config 5ed23df91f done  
Writing manifest to image destination
Storing signatures
cache miss because layer definition was changed
+ busybox --list
+ head -n 10
[
[[
acpid
add-shell
addgroup
adduser
adjtimex
ar
arch
arp
+ echo 'exit 1 to force rebuild'
exit 1 to force rebuild
+ exit 1
error: run commands failed: execute failed: exit status 1
error: exit status 1
```

If you had a local zot repository where with the 'sync' extension enabled and configured in your 'config.yaml, you could do:

    $ stacker build --substitute=DOCKER_LIBRARY=docker://localhost:5900/docker.io/

Upstream zot has a good example of 'sync' configuration in [zot/examples/config-popular-registries.json](https://github.com/project-zot/zot/blob/main/examples/config-popular-registries.json): The specific extension portion of config.yaml to do that would be seen below.

```yaml
extensions:
  sync:
    enable: true
    registries:
      - urls: ["https://docker.io/library"]
        onDemand: true
        tlsVerify: true
        content:
          - prefix: "**"
            destination: "/docker.io/"
```
