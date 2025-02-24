# Using the GitLab-Monitor Chart

The `gitlab-monitor` sub-chart provides Prometheus metrics for GitLab
application-specific data. It talks to PostgreSQL directly to perform
queries to retrieve data for CI builds, pull mirrors, etc. In addition,
it uses the Sidekiq API, which talks to Redis to gather different
metrics around the state of the Sidekiq queues (e.g. number of jobs).

## Requirements

This chart depends on Redis and PostgreSQL services, either as part of
the complete GitLab chart or provided as external services reachable
from the Kubernetes cluster on which this chart is deployed.

## Configuration

The `gitlab-monitor` chart is configured as follows: [Global
Settings](#global-settings) and [Chart Settings](#chart-settings).

## Installation command line options

The table below contains all the possible chart configurations that can be supplied
to the `helm install` command using the `--set` flags.

| Parameter                        | Default               | Description                                    |
| -------------------------------- | --------------------- | ---------------------------------------------- |
| `annotations`                    |                       | Pod annotations                                |
| `enabled`                        | `true`                | gitlab-monitor enabled flag                    |
| `extraContainers`                |                       | List of extra containers to include            |
| `extraInitContainers`            |                       | List of extra init containers to include       |
| `extraVolumeMounts`              |                       | List of extra volumes mountes to do            |
| `extraVolumes`                   |                       | List of extra volumes to create                |
| `image.pullPolicy`               | `IfNotPresent`        | GitLab image pull policy                       |
| `image.pullSecrets`              |                       | Secrets for the image repository               |
| `image.repository`               | `registry.gitlab.com/gitlab-org/build/cng/gitlab-monitor` | gitlab-monitor image repository |
| `image.tag`                      |                       | Unicorn image tag                              |
| `init.image`                     | `busybox`             | initContainer image                            |
| `init.tag`                       | `latest`              | initContainer image tag                        |
| `metrics.enabled`                | `true`                | Toggle Prometheus metrics exporter             |
| `metrics.port`                   | `9168`                | Listen port for the Prometheus metrics exporter       |
| `resources.requests.cpu`         | `50m`                 | gitlab-monitor minimum cpu                            |
| `resources.requests.memory`      | `100M`                | gitlab-monitor minimum memory                         |
| `service.externalPort`           | `9168`                | gitlab-monitor exposed port                           |
| `service.internalPort`           | `9168`                | gitlab-monitor internal port                          |
| `service.name`                   | `gitlab-monitor`      | gitlab-monitor service name                           |
| `service.type`                   | `ClusterIP`           | gitlab-monitor service type                           |

## Chart configuration examples

### image.pullSecrets

`pullSecrets` allows you to authenticate to a private registry to pull images for a pod.

Additional details about private registries and their authentication methods can be
found in [the Kubernetes documentation](https://kubernetes.io/docs/concepts/containers/images/#specifying-imagepullsecrets-on-a-pod).

Below is an example use of `pullSecrets`:

```YAML
image:
  repository: my.unicorn.repository
  pullPolicy: Always
  pullSecrets:
  - name: my-secret-name
  - name: my-secondary-secret-name
```

### annotations

`annotations` allows you to add annotations to the gitlab-monitor pods. For example:

```YAML
annotations:
  kubernetes.io/example-annotation: annotation-value
```

## Global Settings

We share some common global settings among our charts. See the [Globals Documentation](../../globals.md)
for common configuration options, such as GitLab and Registry hostnames.

## Chart Settings

The following values are used to configure the gitlab-monitor pod.

### metrics.enabled

By default, the pod exposes a metrics endpoint at `/metrics`. When
metrics are enabled, annotations are added to each pod allowing a
Prometheus server to discover and scrape the exposed metrics.
