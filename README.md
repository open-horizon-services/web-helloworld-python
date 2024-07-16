# web-helloworld-python

![License](https://img.shields.io/github/license/open-horizon-services/web-helloworld-python)
![Architecture](https://img.shields.io/badge/architecture-x86,arm64-green)
![Contributors](https://img.shields.io/github/contributors/open-horizon-services/web-helloworld-python.svg)

This Open Horizon service demonstrates a simple HTTP server written in Python. The server responds with a "Hello, World!" message on port 8000. To check the "Hello, World!" message in your web browser, navigate to http://localhost:8000/.

## Prerequisites

```sh
NOTE: If you plan to build a new image, a DockerHub login is required and export DOCKER_HUB_ID=[your DockerHub ID] before running installation and Makefile targets.

NOTE: Export the "ARCH" environment variable to set a non-default value for the build process.

To ensure the successful installation and operation of the Open Horizon service, the following prerequisites must be met:

**Open Horizon Management Hub:** To publish this service and register your edge node, you must either [install the Open Horizon Management Hub](https://open-horizon.github.io/quick-start) or have access to an existing hub. You may also choose a downstream commercial distribution like IBM's Edge Application Manager. If you'd like to use the Open Horizon community hub, you may [apply for a temporary account](https://wiki.lfedge.org/display/LE/Open+Horizon+Management+Hub+Developer+Instance) at the Open Horizon community hub, where credentials will be provided.

**Edge Node:**You will need an x86 computer running Linux or macOS, or an ARM64 device such as a Raspberry Pi running Raspberry Pi OS or Ubuntu. The `anax` agent software must be installed on your edge node. This software facilitates communication with the Management Hub and manages the deployment of services.

**Optional Utilities:** Depending on your operating system, you may use:
  - `brew` on macOS
  - `apt-get` on Ubuntu or Raspberry Pi OS
  - `yum` on Fedora
  
  These commands can install `gcc`, `make`, `git`, `jq`, `curl`, and `net-tools`. These utilities are not strictly required but are highly recommended for successful deployment and troubleshooting.


## Installation

1. **Clone the repository:**
    Clone the `web-helloworld-python` GitHub repo from a terminal prompt on the edge node and enter the folder where the artifacts were copied.

   ```shell
   git clone https://github.com/open-horizon-services/web-helloworld-python.git
   cd web-helloworld-python
    ```

2. **Edit Makefile:**
    Adjust the variables at the top of the Makefile as needed, including your Docker ID and unique names for your service and pattern.

    ```shell
    DOCKER_HUB_ID=your_docker_id
    ARCH=amd64
    ```
    You can also override these default values by exporting them in your terminal before running any make commands. This way, you don't have to edit the values directly in the Makefile.
   ```shell
   export DOCKER_HUB_ID=my_docker_id
   export ARCH=my_architecture
   ```
   
    Run `make clean` to confirm that the "make" utility is installed and workin

    Confirm that you have the Open Horizon agent installed by using the CLI to check the version:

    ``` shell
     hzn version
     ```

    It should return values for both the CLI and the Agent (actual version numbers may vary from those shown):

    ``` text
    Horizon CLI version: 2.31.0-1540
    Horizon Agent version: 2.31.0-1540
    ```

    If it returns "Command not found", then the Open Horizon agent is not installed.

    If it returns a version for the CLI but not the agent, then the agent is installed but not running.  You may run it with `systemctl horizon start` on Linux or `horizon-container start` on macOS.

    Check that the agent is in an unconfigured state, and that it can communicate with a hub.  If you have the `jq` utility installed, run `hzn node list | jq '.configstate.state'` and check that the value returned is "unconfigured".  If not, running `make agent-stop` or `hzn unregister -f` will put the agent in an unconfigured state.  Run `hzn node list | jq '.configuration'` and check that the JSON returned shows values for the "exchange_version" property, as well as the "exchange_api" and "mms_api" properties showing URLs.  If those do not, then the agent is not configured to communicate with a hub.  If you do not have `jq` installed, run `hzn node list` and eyeball the sections mentioned above.

    NOTE: If "exchange_version" is showing an empty value, you will not be able to publish and run the service.  The only fix found to this condition thus far is to re-install the agent using these instructions:

    ```shell
    hzn unregister -f # to ensure that the node is unregistered
    systemctl horizon stop # for Linux, or "horizon-container stop" on macOS
    export HZN_ORG_ID=myorg   # or whatever you customized it to
    export HZN_EXCHANGE_USER_AUTH=admin:<admin-pw>   # use the pw deploy-mgmt-hub.sh displayed
    export HZN_FSS_CSSURL=http://<mgmt-hub-ip>:9443/
    curl -sSL https://github.com/open-horizon/anax/releases/latest/download/agent-install.sh | bash -s -- -i anax: -k css: -c css: -p IBM/pattern-ibm.helloworld -w '*' -T 120
    ```

## Usage

### Using the Service Outside of Open Horizon

If you wish to use this service locally for development or testing purposes without integrating with the Open Horizon ecosystem, follow these commands:

```shell

make build
# This command builds the Docker container from your Dockerfile, preparing it for local execution.

make run

```

Test the service:
```sh
make test
```
Stop the running service
```sh

# This runs the container locally. It will start the service on the designated port, making it accessible on your machine.

# Test the service
make test
# This command is used to run any predefined tests that check the functionality of the service. It ensures that the service responds correctly.

make stop
# Stops the running Docker container. Use this command when you are done with testing or running the service locally.
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

### Using the Service Inside Open Horizon
 
 ```shell
docker login
# Log in to your Docker registry where the container image will be pushed.

hzn key create <yourcompany> <youremail>
# This command generates cryptographic keys used to sign and verify the services and patterns you publish to the Open Horizon Management Hub.


make build
# Builds the Docker container from your Dockerfile, similar to the local build process.

make push
```
Publish your service definition and policy, deployment policy files to the Horizon Exchange
```sh
make publish
```

Once it is published, you can get the agent to deploy it:
```sh

# Pushes the built Docker image to your Docker registry, making it available for deployment through Open Horizon.

make publish-service
# Publishes the service to the Open Horizon Management Hub.

make publish-pattern
# Publishes the deployment pattern to the Management Hub.

make agent-run
# Commands the local Open Horizon agent to run the service according to the published pattern.

Then you can watch the agreement form:

```sh
watch hzn agreement list
... (runs forever, so press Ctrl-C when you want to stop)
```
Test the service:
```sh
# Watch agreements and service logs
watch hzn agreement list
# Monitors and displays the agreements between your edge node and the management hub, indicating which services are deployed.

docker ps
# Lists all running Docker containers on your machine, allowing you to see the service container in action.

make test
# Runs tests to ensure the service is operating correctly within the Open Horizon environment.

```sh
make agent-stop
# Stops the Open Horizon agent, effectively undeploying the service from your node.
```

## Advanced Details

### SBoM Service Policy Generation

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

## Usage

To manually run the `web-helloworld-python` service locally as a test, enter `make`.  It will build a container and then run it locally.  This is the equivalent of running `make build` and then `make run`.  Once it successfully builds and runs, you can test it by running `make test` to see the HTML returned from the web server that the container runs.  Entering `docker ps` will show you the `web-helloworld-python` container is running locally.  When you are done and want to stop the container, enter `make stop`.  Entering `docker ps` again will show you that the container is no longer runniing.  Finally, entering `make clean` will remove the image that you built.

To create [the service definition](https://github.com/open-horizon/examples/blob/master/edge/services/helloworld/CreateService.md#build-publish-your-hw), publish it to the hub, and then form an agreement to download and run the service, enter `make publish`.  When installation is complete and an agreement has been formed, exit the watch command with Control-C.  You may then open the web page by entering `make test` or visiting [http://localhost:8000/](http://localhost:8000/) in a web browser.

### All Makefile targets

* `default` - executes the build, and then run targets
* `build` - performs a docker build of the container to create a local image
* `dev` - stops the container if it is running, builds, and then manually runs the container image locally while connectingto a terminal in the container.  Type "exit" to disconnect.
* `run` - stops the container if it is running, then manually runs the container locally
* `check` - populate the service definition with your current environment variables so you can confirm that the actual output matches your intended output
* `test` - request the web page from the web server to confirm that it is running and available
* `push` - Uploads your built container image to DockerHub (assumes you have performed a `docker login` and that your `DOCKER_HUB_ID` variable is set).
* `publish` - Publish the service definition and policy files, and the deployment policy file, to the hub in your organization
* `publish-service` - Publish the service definition file to the hub in your organization
* `remove-service` - Remove the service definition file from the hub in your organization
* `publish-service-policy` - Publish the [service policy](https://github.com/open-horizon/examples/blob/master/edge/services/helloworld/PolicyRegister.md#service-policy) file to the hub in your org
* `remove-service-policy` - Remove the service policy file from the hub in your org
* `publish-deployment-policy` - Publish a [deployment policy](https://github.com/open-horizon/examples/blob/master/edge/services/helloworld/PolicyRegister.md#deployment-policy) for the service to the hub in your org
* `remove-deployment-policy` - Remove a deployment policy for the service from the hub in your org
* `publish-pattern` - Publish the service pattern file to the hub in your organization.  Note: this is a legacy approach and cannot co-exist with any service deployments on the same host.
* `stop` - halt a locally-run container
* `clean` - remove the container image from the local cache
* `agent-run` - register your agent's [node policy](https://github.com/open-horizon/examples/blob/master/edge/services/helloworld/PolicyRegister.md#node-policy) with the hub
* `agent-run-pattern` - register your agent with the hub using the pattern
* `agent-stop` - unregister your agent with the hub, halting all agreements and stopping containers
* `deploy-check` - confirm that a registered agent is compatible with the service and deployment
* `log` - check the agent event logs


### Authors

* [John Walicki](https://github.com/johnwalicki)
* [Troy Fine](https://github.com/t-fine)
___


Enjoy!  Give us [feedback](https://github.com/open-horizon-services/web-helloworld-python/issues) if you have suggestions on how to improve this tutorial.

