# web-helloworld-python
![](https://img.shields.io/github/license/open-horizon-services/web-helloworld-python)
![](https://img.shields.io/badge/architecture-amd%2C%20amd64-green)
![](https://img.shields.io/github/contributors/open-horizon-services/web-helloworld-python)

Extremely simple HTTP server (written in Python) that responds on port 8000 with a hello message.

Begin by editing the variables at the top of the Makefile as desired. If you plan to push it to a Docker registery, make sure you give your docker ID. You may also want to create unique names for your **service** and **pattern** (necessary if you are sharing a tenancy with other users and you are all publishing this service).

To play with this outside of Open Horizon:

```sh
make build
make run
```

Test the service:
```sh
make test
```
Stop the running service
```sh
make stop
```

When you are ready to try it inside Open Horizon:
```sh
docker login
```
Create a cryptographic signing key pair. This enables you to sign services when publishing them to the exchange. This step only needs to be done once.
```sh
hzn key create **yourcompany** **youremail**
```
Build the service:
```sh
make build
make push
```
Publish your service definition and policy, deployment policy files to the Horizon Exchange
```sh
make publish
```

Once it is published, you can get the agent to deploy it:
```sh
make agent-run
```

Then you can watch the agreement form:

```sh
... (runs forever, so press Ctrl-C when you want to stop)
```
Test the service:
```sh
docker ps
make test
```

Then when you are done you can get the agent to stop running it:

```sh
make agent-stop
```

# SBoM Service Policy Generation 

A Software Bill of Materials (SBoM) is a detailed list of components and versions that comprise a piece of software. With software exploints on the rise and open source code being critical to nearly every significant software project today, SBoM education is becoming more and more important. The following steps will lead you through creating an SBoM for the `web-hello-python:1.0.0` image, publish the SBoM data as a service policy, and use the Open-Horizon policy engine to control the deployment of the `web-hello-python` container to an edge node.

1. Create an SBoM for the `web-hello-python:1.0.0` docker image built in the previous section:
```sh
make check-syft
```

2. Generate a service policy from the SBoM data:
```sh
make sbom-policy-gen
```

3. Publish the service and service policy:
```sh
make publish-service
make publish-service-policy
```

4. Publish a deployment policy for the service:
```sh
make publish-deployment-policy
```


