![Flight Deck](https://raw.githubusercontent.com/ten7/flight-deck/main/flightdeck-logo.png)

# Flight Deck Varnish

Flight Deck Varnish is a minimalist Varnish container for Drupal sites on Kubernetes and Docker. You can use it both for local development and production.

Features:
* ConfigMap-friendly YAML configuration

## Tags and versions

There are several tags available for this container, each with different Solr and module support:

| Tags           | Varnish version |
|----------------|-----------------|
| stable, latest | 6.6             |
| edge, develop  | 6.6             |
| x.y.z          | 6.6             |

## Configuration

Instead of a large number of environment variables, this container relies on a file to perform all runtime configuration, `flightdeck-varnish.yml`. Inside the file, create following:

```yaml
---
flightdeck_varnish: {}
```

All configuration is done as items under the `flightdeck_varnish` variable. See the following sections for details as to particular configurations.

You can provide this file in one of three ways to the container:

* Mount the configuration file at path `/config/web/flightdeck-varnish.yml` inside the container using a bind mount, configmap, or secret.
* Mount the config file anywhere in the container, and set the `FLIGHTDECK_CONFIG_FILE` environment variable to the path of the file.
* Encode the contents of `flightdeck-varnish.yml` as base64 and assign the result to the `FLIGHTDECK_CONFIG` environment variable.

### Basic Settings

```yaml
---
flightdeck_varnish:
  secret: "secretish"
  hostPort: "6081"
  controlPort: "6082"
  memSize: "256m"
  storageFile: "/var/lib/varnish/storage.bin"
  storageSize: "1024m"
```

Where:
* **secret** is the Varnish secret used to access the control terminal. Optional.
* **hostPort** is the port on which the Varnish proxy is available. Optional, defaults to `6081`.
* **controlPort** is the port for the Varnish control terminal. Optional, defaults to `6082`.
* **memSize** is the size of memory to dedicate to caching. Optional, defaults to `256m`.
* **storageFile** is the full path in the container to the file to use for longer term varnish caching. Optional, defaults to `/var/lib/varnish/storage.bin`.
* **storageSize** is the size of the storage file. Optional, defaults to `1024m`.

### Specifying VCL

You can specify VCL inline by using the `vcl` key:

```yaml
---
flightdeck_varnish:
  secret: "secretish"
  hostPort: "6081"
  controlPort: "6082"
  memSize: "256m"
  storageFile: "/var/lib/varnish/storage.bin"
  storageSize: "1024m"
  vcl: |
    vcl 4.1;

    import std;

    backend default {
        .host = "127.0.0.1";
        .port = "80";
    }
  ...
```

A default VCL is used by default, based on [Varnish's own example](https://www.varnish-software.com/developers/tutorials/configuring-varnish-drupal/#the-drupal-vcl-file).

### Using an external file

Sometimes you may wish to use an external file instead of an inline VCL. You can do this with `vclPath`:

```yaml
---
flightdeck_varnish:
  secret: "secretish"
  hostPort: "6081"
  controlPort: "6082"
  memSize: "256m"
  storageFile: "/var/lib/varnish/storage.bin"
  storageSize: "1024m"
  vclPath: "/config/vcl/flightdeck.vcl"
```

Where:
* **vclPath** is the path inside the container to the VCL file.

This method is particularly useful on Kubernetes as it allows you to keep your VCL in a separate Configmap which may be easily edited. Furthermore, multiple files may be stored within the configmap as needed.

## Part of Flight Deck

This container is part of the [Flight Deck](https://flightdeck.ten7.com) library of containers for Drupal local development and production workloads on Docker, Swarm, and Kubernetes.

Flight Deck is used and supported by [TEN7](https://ten7.com/).

## Debugging

If you need to get verbose output from the entrypoint, set `flightdeck_debug` to `true` or `yes` in the config file.

```yaml
---
flightdeck_debug: yes
```

This container uses [Ansible](https://www.ansible.com/) to perform start-up tasks. To get even more verbose output from the start up scripts, set the `ANSIBLE_VERBOSITY` environment variable to `4`.

If the container will not start due to a failure of the entrypoint, set the `FLIGHTDECK_SKIP_ENTRYPOINT` environment variable to `true` or `1`, then restart the container.

## License

Flight Deck is licensed under GPLv3. See [LICENSE](https://raw.githubusercontent.com/ten7/flight-deck/master/LICENSE) for the complete language.
