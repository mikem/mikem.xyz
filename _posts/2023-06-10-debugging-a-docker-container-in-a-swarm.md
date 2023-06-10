---
layout: post
title: Debugging a Docker Container in a Swarm
---

Let's say you have a container that's being launched by something which reads a [Docker Compose file](https://docs.docker.com/compose/compose-file/). For some reason the container isn't launching correctly, it's crashlooping, which makes it difficult to a) read the logs in time, and b) log in to investigagte.

There are two Compose file attributes for the service which might come in useful:

[command](https://docs.docker.com/compose/compose-file/05-services/#command): this will override the comamnd specified in the Dockerfile. We need something which will not immediately terminate; `tail -f /dev/null` will work nicely.

[healthcheck](https://docs.docker.com/compose/compose-file/05-services/#healthcheck): if the container has a healthcheck configured, it will restart when the healthcheck timeout expires. Since the container is broken, this is likely going to happen while you're debugging, which can be annoying. Thankfully this can be disabled.

The service definition in the Compose file then looks like this:

```yaml
service:
  foo:
    image: example/foo:latest
    # all the other usual settings
    command: "tail -f /dev/null"
    healthcheck:
      disable: true
```

Re-deploy the containers and debug in peace. Don't forget to remove those lines when you're done.

# References

- [How to Keep Docker Container Running for Debugging](https://devopscube.com/keep-docker-container-running/) at DevopsCube
- [Docker Compose Specification Overview](https://docs.docker.com/compose/compose-file/)
