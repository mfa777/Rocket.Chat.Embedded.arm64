# Rocket.Chat docker image for 64bit ARM servers

You can edit the following line in `docker-compose.yml` file to change the version of Rocket.Chat the image will contain:

```
ENV RC_VERSION 4.1.2
```

To build this image and publish to your local registry, use the command:

```
docker build -t rocketchat:arm64-4.1.2 .
```
