# How to run Docker in Docker with Rancher Desktop in Mac

Usually you will see instructions to do the following:
```
docker run --entrypoint sh -it --privileged -v $HOME/.rd/docker.sock:/var/run/docker.sock docker:dind
```

You tried it on your host machine butthis will not make it (don't ask me why, I have no idea). Try this instead:

```bash
➜  ~ rdctl shell                       # get in to the virtual machine first
lima-rancher-desktop:/Users/cuichli$ docker run -it --entrypoint sh -v /var/run/docker.sock:/var/run/docker.sock --privileged docker:dind
```

Then in the docker container you just created, you will be able to connect to the docker daemon. 