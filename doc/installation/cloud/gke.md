# Preparing GKE resources

For a fully functional GitLab instance, you will need a few resources before
deploying the `gitlab` chart. The following is how these charts are deployed
and tested within GitLab.

NOTE: **Note:** Google provides a whitepaper for [deploying production-ready GitLab on
Google Kubernetes Engine][whitepaper], including all steps and external
resource configuration. These are alternative to this document, and the
deployed chart will behave slightly differently. For example, the default
domain is configured with [xip.io](http://xip.io), which may experience issues due to [rate limiting](https://letsencrypt.org/docs/rate-limits/) with
Let's Encrypt.

[whitepaper]: https://cloud.google.com/solutions/deploying-production-ready-gitlab-on-gke

## Creating the GKE cluster

To get started easier, a script is provided to automate the cluster creation.
Alternatively, a cluster can be created manually as well.

### Scripted cluster creation

A [bootstrap script](https://gitlab.com/charts/gitlab/blob/master/scripts/gke_bootstrap_script.sh)
has been created to automate much of the setup process for users on GCP/GKE.

The script will:

1. Create a new GKE cluster.
1. Allow the cluster to modify DNS records.
1. Setup `kubectl`, and connect it to the cluster.
1. Initialize Helm and install Tiller.

Google Cloud SDK is a dependency of this script, so make sure it's
[set up correctly](../tools.md#connecting-to-the-gke-cluster) in order for the script
to work.

The script reads various parameters from environment variables and an argument
`up` or `down` for bootstrap and clean up respectively.

The table below describes all variables.

| Variable        | Description                                                                 | Default value                    |
|-----------------|-----------------------------------------------------------------------------|----------------------------------|
| REGION          | The region where your cluster lives                                         | us-central1                      |
| ZONE            | The zone where your cluster instances lives                                 | us-central1-a                    |
| CLUSTER_NAME    | The name of the cluster                                                     | gitlab-cluster                   |
| CLUSTER_VERSION | The version of your GKE cluster                                             | GKE default, check the [GKE release notes](https://cloud.google.com/kubernetes-engine/release-notes) |
| MACHINE_TYPE    | The cluster instances' type                                                 | n1-standard-4                    |
| NUM_NODES       | The number of nodes required.                                               | 2                                |
| PROJECT         | the id of your GCP project                                                  | No defaults, required to be set. |
| RBAC_ENABLED    | If you know whether your cluster has RBAC enabled set this variable.        | true                             |
| PREEMPTIBLE     | Cheaper, clusters live at *most* 24 hrs. No SLA on nodes/disks              | false                            |
| USE_STATIC_IP   | Create a static IP for Gitlab instead of an ephemeral IP with managed DNS   | false                            |
| INT_NETWORK     | The IP space to use within this cluster                                     | default                          |

Run the script, by passing in your desired parameters. It can work with the
default parameters except for `PROJECT` which is required:

```bash
PROJECT=<gcloud project id> ./scripts/gke_bootstrap_script.sh up
```

The script can also be used to clean up the created GKE resources:

```bash
PROJECT=<gcloud project id> ./scripts/gke_bootstrap_script.sh down
```

With the cluster created, continue to [creating the DNS entry](#dns-entry).

### Manual cluster creation

Two resources need to be created in GCP, a Kubernetes cluster and an external IP.

#### Creating the Kubernetes cluster

To provision the Kubernetes cluster manually, follow the
[GKE instructions](https://cloud.google.com/kubernetes-engine/docs/how-to/creating-a-container-cluster).

- We recommend a cluster with 8vCPU and 30GB of RAM.
- Make a note of the cluster's region, it will be needed in the following step.

#### Creating the external IP

An external IP is required so that your cluster can be reachable. The external
IP needs to be regional and in the same region as the cluster itself. A global
IP or an IP outside the cluster's region will **not work**.

To create a static IP run:

`gcloud compute addresses create ${CLUSTER_NAME}-external-ip --region $REGION --project $PROJECT`

To get the address of the newly created IP:

`gcloud compute addresses describe ${CLUSTER_NAME}-external-ip --region $REGION --project $PROJECT --format='value(address)'`

We will use this IP to bind with a DNS name in the next section.

## DNS Entry

If you created your cluster manually or used the `USE_STATIC_IP` option with the scripted creation,
you'll need a public domain with an A record wild card DNS entry pointing to the IP we just created.

Follow the [Google DNS quickstart guide](https://cloud.google.com/dns/quickstart)
to create the DNS entry.

## Next Steps

Continue with the [installation of the chart](../deployment.md) once you have
the cluster up and running, and the static IP and DNS entry ready.
