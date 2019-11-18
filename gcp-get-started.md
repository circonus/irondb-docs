# Getting Started With Google Cloud Marketplace

Circonus IRONdb is available on the Google Cloud Marketplace as a multi-VM
(virtual machine) solution.  The solution configures your chosen number of
nodes into a cluster, suitable for either standalone use or as a private
storage cluster connected to the Circonus SaaS platform.

## Deploying the Circonus IRONdb GCP Solution

Access the
[Circonus IRONdb
page](https://console.cloud.google.com/marketplace/details/circonus-public/irondb)
in Google Marketplace and click the **LAUNCH ON COMPUTE ENGINE** button.

![Image: 'gcp_gs_launch001.png'](/assets/gcp_gs_launch001.png?raw=true)

In the "New Circonus IRONdb deployment" window, select the desired Zone,
Instance Count, and Machine Type. Circonus strongly recommends using SSD for
the Metric data disk (the volume where your telementry data will be stored on
each node). Choose the desired size of the Metric data disk.

If you wish to send Graphite metrics to the cluster from outside the Google
Cloud, leave the "Allow TCP port 2003" firewall rule enabled.

If you wish to send OpenTSDB metrics to the cluster from outside the Google
Cloud, leave the "Allow TCP port 4242" firewall rule enabled.

By default, Circonus IRONdb will report software malfunctions to Circonus. This
requires internet access from each node of the cluster. If you do not wish to
allow this outbound traffic, uncheck the "Automated crash reporting" option.

![Image: 'gcp_gs_new_deploy001.png'](/assets/gcp_gs_new_deploy001.png?raw=true)

Click the **Deploy** button. The Deployment Manager page provides the status of
the deployment.

![Image: 'gcp_gs_deploy_mgr001.png'](/assets/gcp_gs_deploy_mgr001.png?raw=true)

Once the deployment succeeds, you can check the status of Circonus IRONdb on
each cluster node by logging into the VM and running the following command:

```
# /usr/bin/systemctl status circonus-irondb
```

Your Circonus IRONdb deployment is now ready to accept metric data. For more
information on how to send data in, see the [Integrations
page](https://www.irondb.io/docs/integrations.html) in the Circonus IRONdb
documentation.

If you plan to connect this deployment to your Circonus SaaS account, see below
for additional steps.

## Connecting A Deployment to Circonus SaaS

Connecting a GCP Marketplace deployment as a swimlane for a Circonus SaaS
account requires static IP address assignments for the nodes, as well as
opening the IRONdb API port (8112/tcp) to the SaaS egress IPs, which can be
obtained by querying the DNS name `out.circonus.net`:

```
$ dig out.circonus.net +short
35.184.18.2
104.198.179.151
```

After deploying a Marketplace cluster, navigate to your project's "VPC network"
page:

```
https://console.cloud.google.com/networking/firewalls/list?project=YOUR-PROJECT-ID-HERE
```

![Image: 'gcp_gs_firewall_rules001.png'](/assets/gcp_gs_firewall_rules001.png?raw=true)

### Change To Static IPs

1. Choose "External IP addresses" from the left menu.
![Image: 'gcp_gs_ext_ip001.png'](/assets/gcp_gs_ext_ip001.png?raw=true)
1. Locate the address resources associated with the IRONdb cluster (the **In
use by** column).
1. For each address identified, change the Type from "Ephemeral" to "Static".
This will pop a dialog where you must enter a Name and, optionally, a
Description. **Note: the values cannot be changed once set.**

![Image: 'gcp_gs_static001.png'](/assets/gcp_gs_static001.png?raw=true)

#### Gather Node UUIDs

Circonus will use the node UUIDs (assigned internally during deployment) to
create DNS records that will be used to connect to the cluster from the
Circonus SaaS environment.

Log into one of the cluster nodes and run the following command:

```
grep gcpmkt /etc/hosts
```

The output will be a list of IP addresses and hostnames that start with a UUID.
Send this list, along with the external IP address that corresponds to each
node, to Circonus.

### Add Firewall Rule

Choose "Firewall rules" from the left menu, then click the `+ CREATE FIREWALL
RULE` option at the top of the page.

![Image: 'gcp_gs_fw_add_rule001.png'](/assets/gcp_gs_fw_add_rule001.png?raw=true)


1. Enter a descriptive name for the rule, such as `circonus-saas-access`.
1. (optional) Add a longer description.
1. Select which Network the rule applies to. Often this is `default` but choose
the one that matches where the cluster is deployed.
1. `Priority` can be left at its default.
1. Leave `Direction of traffic` and `Action on match` at their defaults (Ingress and Allow,
respectively)
1. For `Targets` select `Specified target tags`
1. In the `Target tags` field, enter the same tag used for the port 2003/4242
rules, which is `[deployment_name]-db-tier`.
1. For `Source filter` choose `IP ranges`.
1. In the `Source IP ranges` field enter, one at a time, `35.184.18.2/32`, `104.198.179.151/32`
(or change the IPs if different from the DNS result above.)
1. Leave `Second source filter` as `None`.
1. For `Protocols and ports`, the default is `Specified protocols and port`.
Check the `tcp` box and enter `8112` in the input field next to it.
1. Click **Create**.

## Open Source Licenses

License information for all open source software included in or distributed
with Circonus IRONdb may be found on any cluster node in the directory,
`/opt/circonus/share/licenses/`.
