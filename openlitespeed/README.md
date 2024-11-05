OpenLiteSpeed
===============
[OpenLiteSpeed](https://github.com/litespeedtech/openlitespeed) is the Open Source edition of [LiteSpeed Web Server Enterprise](https://www.litespeedtech.com/). 
More information about OpenLiteSpeed can be found [here](https://openlitespeed.org/).

## Purpose 
This directory contains a Dockerfile that builds [OpenLiteSpeed](https://github.com/litespeedtech/openlitespeed) with [OQS-BoringSSL](https://github.com/open-quantum-safe/boringssl), which allows OpenLiteSpeed to negotiate quantum-safe key exchange using [liboqs](https://github.com/open-quantum-safe/liboqs/).



## Getting started

## Server
### Building
Assuming Docker is [installed](https://docs.docker.com/install) the following command

```
docker build -t lsws -f Dockerfile-server .
docker network create lsws-test
docker run --network lsws-test --name lsws -it lsws bash
```

will run the container for the quantum-safe crypto (QSC) protected OpenLiteSpeed server on the docker network called lsws-test.

### Usage
Documentation for using the server docker image is contained in the separate [USAGE-server.md](USAGE-server.md) file.

## Client

The [curl QUIC demo](https://github.com/open-quantum-safe/oqs-demos/blob/main/curl/README-QUIC.md) can be used to test the post quantum key exchange.

The following command

```
docker run --network lsws-test --name curl-quic-instance -it openquantumsafe/curl-quic bash
```

runs the container with a QSC-enabled QUIC curl on the same network as the server.
### Usage
Documentation for using the client docker image is contained in the separate [USAGE-client.md](USAGE-client.md) file.


## Disclaimer

[THIS IS NOT FIT FOR PRODUCTION USE](https://github.com/open-quantum-safe/openssl#limitations-and-security).
