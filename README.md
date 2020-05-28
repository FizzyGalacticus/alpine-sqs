# Alpine SQS _(alpine-sqs)_

> Dockerized ElasticMQ server over Alpine Linux for local development.

Alpine SQS provides a containerized Java implementation of the Amazon Simple Queue Service (AWS-SQS). It is based on ElasticMQ running Alpine Linux and the OpenJDK Java 8 JRE. It is compatible with AWS's API, CLI as well as the Amazon Java SDK. This allows for quicker local development without having to incurr in infrastructure costs.

The goal of this repository is to maintain an updated Docker environment for ElasticMQ with an integrated web UI for visualizing queues and messages.

## Table of Contents

- [Background](#background)
- [Install](#install)
- [Usage](#usage)
- [Maintainer](#maintainer)
- [Contribute](#contribute)
- [License](#license)

## Background
When searching for existing local implementations of SQS I came across a Docker image by [@vsouza](https://github.com/vsouza) called [docker-SQS-local](https://github.com/vsouza/docker-SQS-local) with over 11K pulls at the time.

This introduced me to ElasticMQ, which this project is based on and is described by it's creators as:

> a  message queue system, offering an actor-based Scala and an SQS-compatible REST (query) interface.

Using his work as inspiration I decided to improve upon it by implementing the following:

- Reduce the Docker image foot-print as much as possible.
- Automatically update to the latest ElasticMQ server.
- Automatic tests & builds (work in progress).
- Thorough documentation.

### See also
For more information on the different projects this work is based on, please visit:

- [ElasticMQ](https://github.com/softwaremill/elasticmq).

## Install
### Pre-requisites

To be able to use this environment, please make sure you have installed the latest version of [Docker](https://docs.docker.com/engine/installation/).

If you intend to build the environment yourself, it is recommended that you also install the latest version of [Docker Compose](https://docs.docker.com/compose/install/).

### Installation methods
You can obtain the environment in two ways; The easiest is to pull the image directly from Docker Hub. Also, you may clone this repository and build/run it using Docker Compose.

#### 2. Building from scratch
```
git clone https://github.com/FizzyGalacticus/alpine-sqs
```
## Usage
### Running the environment
Depending on how you chose to install the environment, you can initialize it in three ways:

#### 1. `docker run` method
Use this method if you're pulling directly from Docker Hub and do not have a `docker-compose.yml` file.

```
docker run --name alpine-sqs -p 9324:9324 -p 9325:9325 -d roribio16/alpine-sqs:latest
```

Custom configuration files may be used to override default behaviors. You can mount a volume mapping the container's `/opt/custom` directory to a folder in your host machine where the custom configuration files are stored.

Providing for sake of example that in your host machine directory `/opt/alpine-sqs` you have both `elasticmq.conf` and `sqs-insight.conf` files, you can run the container with:

```
docker run --name alpine-sqs -p 9324:9324 -p 9325:9325 -v /opt/alpine-sqs:/opt/custom -d roribio16/alpine-sqs:latest
```

For any configuration file not explicitly included in the container's `/opt/custom` directory, `alpine-sqs` will fall back to using the default configuration files listed [here](https://github.com/FizzyGalacticus/alpine-sqs/tree/master/opt).

#### 2. `docker-compose up` method
If you've cloned the repository you can still take advantage of the image present in Docker Hub by running the container from the default `docker-compose.yml` file. This will pull the pre-built image from the public registry and run it with the same values stated in the previous method.

```
docker-compose up -d
```

#### 3. `docker-compose up --build` method
To build the image from scratch and then run the corresponding container, use this method.

```
docker-compose -f docker-compose.build up -d --build
```

> **Note**: To use any of the Docker Compose methods, you need to clone this repository as well as have Docker Compose installed.

>> **Note 2**: Depending on your platform, you may need to adjust how you declare mounted volumes. You can find instructions for your specific platform [here](https://github.com/roribio/alpine-sqs/wiki/Sharing-files-with-host-machine).

### Working with queues
ElasticMQ provides an Amazon-SQS compatible interface. This means you may use the AWS command-line tool, API calls and the Java SDK, to interact with local queues the same as if interacting with the actual SQS.

#### Default queue
The default configuration provisions ElasticMQ with a initial queue of the same name at run time. This allows you to start pushing messages to the queue without further configuration.

To make use of this queue, point your client to: `http://localhost:9324/queue/default`.

#### Sending a message
To send messages to a queue you need to specify the new endpoint url and queue url along with the message payload. The following example uses the AWS CLI to send a message to the `default` queue.

```
aws --endpoint-url http://localhost:9324 sqs send-message --queue-url http://localhost:9324/queue/default --message-body "Hello, queue!"
```

#### Viewing messages

You can poll for messages from the command-line like so:

```
aws --endpoint-url http://localhost:9324 sqs receive-message --queue-url http://localhost:9324/queue/default --wait-time-seconds 10
```

### Creating new queues
You can create new queues by using the command-line or configuring ElasticMQ directly.

##### AWS CLI
```
aws --endpoint-url http://localhost:9324 sqs create-queue --queue-name newqueue
```

##### Edit ElasticMQ configuration file
Navigate to the directory where the configuration files reside and edit the `elasticmq.conf` file to add a new entry for each queue to the `queue` block.

```
queues {
    default {
        defaultVisibilityTimeout = 10 seconds
        delay = 5 seconds
        receiveMessageWait = 0 seconds
    },
    newqueue {
        defaultVisibilityTimeout = 10 seconds
        delay = 5 seconds
        receiveMessageWait = 0 seconds
    }
}
```

> **Note**: The configuration directory location inside the container is located at `/opt/config`. If you mounted that volume onto your host, you can also find the configuration files there.

After editing the `elasticmq.conf` file, you need to restart the ElasticMQ server by running the `supervisorctl restart elasticmq` command inside the container. If you're editing the configuration file outside of the container, use this command:

```
docker exec -it alpine-sqs sh -c "supervisorctl restart elasticmq"
```

> Consult the [AWS CLI Command Reference](http://docs.aws.amazon.com/cli/latest/reference/sqs/index.html#cli-aws-sqs) or the [AWS SDK for Java](http://docs.aws.amazon.com/sdk-for-java/v1/developer-guide/examples-sqs-message-queues.html) guide for more examples and information.

## Contribute
PRs are accepted and encouraged!

Please direct any questions, requests, or comments to the [Issues](https://github.com/FizzyGalacticus/alpine-sqs/issues) section of this project.

## License
TBD -- [Awaiting response from original author](https://github.com/roribio/alpine-sqs/issues/31)
