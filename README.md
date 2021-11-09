![Flightdeck](https://raw.githubusercontent.com/ten7/flight-deck/master/flightdeck-logo.png)

# Flightdeck Zookeeper

Flightdeck Zookeeper is a minimalist Zookeeper container for Drupal sites on Kubernetes and Docker. You can use it both for local development and production.

Features:
* ConfigMap-friendly YAML configuration
* Supports ensemble mode

## Tags and versions

There are several tags available for this container, each with different Zookeeper and Drupal module support:

| Tags | Zookeeper Version |
| ---- | ----------------- |
| latest | 3.7 |
| x.y.z | 3.7 |


## Configuration

Instead of a large number of environment variables, this container relies on a file to perform all runtime configuration, `flightdeck-zookeeper.yml`. Inside the file, create following:

```yaml
---
flightdeck_zookeeper: {}
```

All configuration is done as items under the `flightdeck_zookeeper` variable. See the following sections for details as to particular configurations.

You can provide this file in one of three ways to the container:

* Mount the configuration file at path `/config/zookeeper/flightdeck-zookeeper.yml` inside the container using a bind mount, configmap, or secret.
* Mount the config file anywhere in the container, and set the `FLIGHTDECK_CONFIG_FILE` environment variable to the path of the file.
* Encode the contents of `flightdeck-zookeeper.yml` as base64 and assign the result to the `FLIGHTDECK_CONFIG` environment variable.

### Basic settings

```yaml
---
flightdeck_zookeeper:
  tickTime: "1000"
  dataDir: "/data"
  clientPort: "2181"
```

Where:
* **tickTime** is the basic time unit in milliseconds used by ZooKeeper. Optional, defaults to `1000`.
* **dataDir** is the directory to store Zookeeper files. Optional, defaults to `/data`.
* **clientPort** is the port on which to listen for client connections. Optional, defaults to `2181`.

### Zookeeper ensemble

Often, you will want to run multiple Zookeeper instances as an *ensemble*, or a redundant cluster of instances. There are several settings needed to configure an ensemble for this container:

```yaml
---
flightdeck_zookeeper:
  initLimit: "120"
  syncLimit: "120"
  myId: "1"
  myIdFromHostname: false
  servers:
    - hostname: "zookeeper-0"
      quorumPort: 2888
      leaderPort: 3888
    - hostname: "zookeeper-1"
    - hostname: "zookeeper-2"
```

Where:
* **initLimit** is the amount of time in ticks (see `tickTime`) for a Zookeeper ensemble to reach a quorum. Optional, defaults to `120`.
* **syncLimit** is the amount of time in ticks (see `tickTime`) a Zookeeper follower instance can be out of sync from the Zookeeper leader instance. Optional, defaults to `120`.
* **myId** is the server ID number greater than 0. Required unless `myIdFromHostname` is `true`.
* **myIdFromHostname** instructs the container to attempt to extract the value of `myid` from the container hostname. Optional, defaults to `true`.
* **servers** is a list of items specifying the Zookeeper `host`, the `quorumPort` (default `2888`), and the `leaderPort` (default `3888`).

The `myIdFromHostname` setting is particularly useful when deploying this container as a Statefulset in Kubernetes, as hostnames end in `-n` where *n* is a zero indexed replica number. Whatever number extracted from the hostname is incremented by one (0 -> 1) so as to meet Zookeeper `myid` requirements.

### Autopurging files

Zookeeper retains snapshots of the `dataDir`. You can control this behavior under the `autopurge` key:

```yaml
---
flightdeck_zookeeper:
  autopurge:
    state: true
    snapRetainCount: 3
    purgeInterval: 24
```

Where:
* **state** is if snapshot autopurging is enabled. Optional, defaults to `true`.
* **snapRetainCount** is the number of snapshots to retain. Optional, defaults to `3`.
* **purgeInterval** is the time in hours to purge previous snapshots. Optional, defaults to `24`.

## Using with Docker Compose

Create the `flightdeck-zookeeper.yml` file relative to your `docker-compose.yml`. Define the `Zookeeper` service mounting the file as a volume:

```yaml
version: '3'
services:
  Zookeeper:
    image: ten7/flight-deck-Zookeeper
    volumes:
      - ./flightdeck-zookeeper.yml:/config/Zookeeper/flightdeck-zookeeper.yml
    ports:
      - "8003:8983"
```

## Part of Flightdeck

This container is part of the [Flightdeck library](https://github.com/ten7/flight-deck) of containers for Drupal local development and production workloads on Docker, Swarm, and Kubernetes.

Flightdeck is used and supported by [TEN7](https://ten7.com/).


## Debugging

If you need to get verbose output from the entrypoint, set `flightdeck_debug` to `true` or `yes` in the config file.

```yaml
---
flightdeck_debug: yes
```

This container uses [Ansible](https://www.ansible.com/) to perform start-up tasks. To get even more verbose output from the start up scripts, set the `ANSIBLE_VERBOSITY` environment variable to `4`.

If the container will not start due to a failure of the entrypoint, set the `FLIGHTDECK_SKIP_ENTRYPOINT` environment variable to `true` or `1`, then restart the container.

## License

Flightdeck is licensed under GPLv3. See [LICENSE](https://raw.githubusercontent.com/ten7/flight-deck/master/LICENSE) for the complete language.
