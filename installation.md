# IRONdb Installation

## System Requirements

IRONdb requires one of the following operating systems:
* [OmniOS](https://omnios.omniti.com/), version r151014.
* RHEL/CentOS, version 7.x.
* Ubuntu 16.04 LTS.

Additionally, IRONdb requires the [ZFS](http://open-zfs.org/) filesystem. This
is available natively on OmniOS and Ubuntu, but for EL7 installs, you will need
to obtain ZFS from the [ZFS on
Linux](https://github.com/zfsonlinux/zfs/wiki/Getting-Started)
project. The setup script expects a zpool to exist, but you do not need to
create any filesystems or directories ahead of time. Please refer to the
appendix [ZFS Setup Guide](/zfs-guide.md) for details and examples.

Hardware requirements will necessarily vary depending upon system scale and cluster size. Please [contact us](./contact.md) with questions regarding system sizing. Circonus recommends the following minimum system specification for the single-node, free, 25K-metrics option:

* 1 CPU
* 4 GB RAM
* SSD-based storage, 20 GB available space

The following network protocols and ports are utilized. These are defaults and
may be changed via configuration files.

* 2003/tcp (Carbon plaintext submission)
* 8112/tcp (admin UI, HTTP REST API, [cluster replication](./operations.md#replication), [request proxying](./operations.md#proxying))
* 8112/udp ([cluster gossip](./operations.md#replication))

## Installation Steps

Follow these steps to get IRONdb installed on your system. If you are using one of our pre-built Amazon EC2 images, **these steps are already done for you**, and your free-25K instance will be configured automatically on first boot. Please refer to [EC2 installation](#ec2-installation) below.

System commands must be run as a privileged user, such as `root`, or via `sudo`.

Configure package repositories. During the IRONdb beta period, our development
(aka "pilot") repo is required.

(OmniOS) Add the Circonus IPS package publisher:

    /usr/bin/pkg set-publisher \
      -g http://updates.circonus.net/omnios/r151014/ \
      -g http://pilot.circonus.net/omnios/r151014/ circonus

(EL7) Create the file `/etc/yum.repos.d/Circonus.repo` with the following contents:

    [circonus]
    name=Circonus
    baseurl=http://pilot.circonus.net/centos/7/x86_64/
    enabled = 1
    gpgcheck = 0
    metadata_expire = 5m

    [circonus-crash-reporting]
    name=Circonus - Crash Reporting
    baseurl=http://updates.circonus.net/backtrace/centos/el7/
    enabled = 1
    gpgcheck = 0

(Ubuntu 16.04) Create the file `/etc/apt/sources.list.d/circonus.list` with the
following contents:

    deb http://pilot.circonus.net/ubuntu/ xenial main

Install the package:
* (OmniOS) `/usr/bin/pkg install pkg:/platform/irondb`
* (EL7) `/usr/bin/yum install circonus-platform-irondb`

We have a helper package on Ubuntu that works around issues with dependency
resolution, since IRONdb is very specific about the versions of dependent
Circonus packages, and apt-get is unable to cope with them. The helper package
must be installed first, i.e., it cannot be installed in the same transaction
as the main package.
```
/usr/bin/apt-get install circonus-platform-irondb-apt-policy
/usr/bin/apt-get install circonus-platform-irondb
```

Prepare site-specific information for setup. These values may be set via shell environment variables, or as arguments to the setup script. The environment variables are listed below.
   * ##### IRONDB\_NODE\_UUID

     *\(required\)* The ID of the current node, which must be unique within a given cluster. You may use the `uuidgen` command that comes with your OS, or generate a UUID with an external tool or website.
   * ##### IRONDB\_NODE\_ADDR

     *\(required\)* The IPv4 address of the current node, e.g., "192.168.1.100".
   * ##### IRONDB\_CHECK\_UUID

     *\(required\)* Check ID for Graphite metric ingestion, which must be the same on all cluster nodes. You may use the `uuidgen` command that comes with your OS, or generate a UUID with an external tool or website.
   * ##### IRONDB\_CHECK\_NAME

     *\(required\)* The string that will identify Graphite-compatible metrics stored in the check identified by `IRONDB_CHECK_UUID`. For example, if you submit a metric named "my.metric.1", and the check is named "test", the resulting metric name in IRONdb will be "graphite.test.my.metric.1".
   * ##### IRONDB\_CRASH\_REPORTING

     *\(optional\)* Control enablement of automated crash reporting. Default is "on". IRONdb utilizes sophisticated crash tracing technology to help diagnose errors. Enabling crash reporting requires that the system be able to connect out to the Circonus reporting endpoint: https://circonus.sp.backtrace.io:6098 . If your site's network policy forbids this type of outbound connectivity, set the value to "off".
   * ##### IRONDB\_ZPOOL

     *\(optional\)* The name of the zpool that should be used for IRONdb storage. If this is not specified and there are multiple zpools in the system, setup chooses the pool with the most available space.

Run the setup script. All required options must be present, either as environment variables or via command-line arguments. A mix of environment variables and arguments is permitted, but environment variables take precedence over command-line arguments. Use the `-h` option to view a usage summary:

    Usage: /opt/circonus/bin/setup-irondb [-h] -a <ip-address> -n <node-uuid> -c <check-name> -u <check-uuid>
           [-b (on|off)] [-z <zpool>]
      -a <ip-address>  : Local IP address to use
      -n <node-uuid>   : Local node UUID
      -c <check-name>  : Graphite check name
      -u <check-uuid>  : Graphite check UUID
      -b on|off        : Enable/disable crash reporting (default: on)
      -z <zpool>       : Use this zpool for data storage
                         (default: choose pool with most available space)
      -h               : Show usage summary

The setup script will configure your IRONdb instance and start the service. Upon successful completion, it will print out specific information about how to submit Graphite metrics. IRONdb supports both Carbon plaintext submission (port 2003) or HTTP POST. See the [Graphite Ingestion](./graphite-ingestion.md) section for details.

Obtain your license information from your Circonus account profile: https://YOURACCOUNT.circonus.com/profile
* If you do not have an IRONdb license, click the `+` to the right of the Licenses section of the account profile page to add a new license.
* If you do not have a Circonus account, [sign up for free](https://login.circonus.com/signup).

Add the `<license>` stanza from your chosen IRONdb license to the file `/opt/circonus/etc/licenses.conf` on your IRONdb instance, within the enclosing `<licenses>` tags. It should look something like this:

    <licenses>
      <license id="(number)" sig="(cryptographic signature)">
        <graphite>true</graphite>
        <max_streams>25000</max_streams>
        <company>MyCompany</company>
      </license>
    </licenses>

Restart the IRONdb service:
* (OmniOS) `/usr/sbin/svcadm restart irondb`
* (EL7, Ubuntu) `/bin/systemctl restart circonus-irondb`

## EC2 Installation

Circonus makes available EC2 AMIs that come preinstalled with Ubuntu and
IRONdb. The first time an instance of the AMI boots, the setup script runs and
configures a standalone instance. The AMI is suitable for HVM instance types, and the
minimum recommended instance type is `m4.large`.

For clusters, choose an instance type that has enough vCPUs to handle both incoming data and replicating data to other cluster nodes. A simple formula for a cluster of `N` nodes would be `N+1` vCPUs, up to maximum of 16.

Since the 0.6 beta release, IRONdb images are named in the format `irondb-VERSION`, where `VERSION` is the product version. Older versions of the AMI had `-single` appended to the name. There is no difference in how the initial automated setup works. The initial single-node configuration may be reconfigured for clustering prior to ingesting any metric data.

You can find IRONdb AMIs by
[searching](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/finding-an-ami.html)
for "IRONdb" among the community AMIs, using either the EC2 Console or the AWS
CLI.

For example, using the [AWS CLI](https://aws.amazon.com/cli/), you could locate
available IRONdb AMIs in the `us-west-1` region:

    aws ec2 describe-images \
      --region us-west-1 \
      --filters 'Name=description,Values=*IRONdb*' \
      --query 'Images[*].{ID:ImageId,NAME:Name}' 

Circonus currently publishes AMIs to the following regions:
* ap-northeast-1 (Tokyo)
* eu-central-1 (Frankfurt)
* us-east-1 (N. Virginia)
* us-east-2 (Ohio)
* us-west-1 (N. California)
* us-west-2 (Oregon)

Setup expects the required options to be provided as [instance user-data](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-metadata.html#instancedata-add-user-data). When launching your instance, add the necessary options in environment-variable format, substituting your own UUIDs and check name for the sample ones:

    IRONDB_NODE_UUID="12345678-9abc-def0-1234-456789000000"
    IRONDB_CHECK_NAME="test"
    IRONDB_CHECK_UUID="98765432-1abc-def9-8765-432100000000"
    IRONDB_CRASH_REPORTING="on"

The setup process will detect the local IP address of the instance at boot, so it is not necessary to specify `IRONDB_NODE_ADDR` in the user-data \(it will be ignored even if present.\)

Setup also expects to configure a zpool on the secondary EBS volume that is specified by the AMI. You do not need to specify `IRONDB_ZPOOL` in the user-data.

If you do not wish to use the pre-built AMI, you will need to create an
instance yourself using one of the supported operating systems and follow the
full setup instructions in the [previous section](#installation-steps).

### EC2 Security Groups

At a minimum, the nodes in an IRONdb cluster need to communicate with one another using port 8112, over both TCP and UDP. The TCP transport is used for replicating data,
while UDP is used for exchanging state information about each node, known as "gossip".

### EC2 CLI Example

The following is an example of using the [AWS command-line client](https://aws.amazon.com/cli/) to launch an instance with IRONdb user data. The user data was pasted into a local file, named `my-user-data` in this example:

    aws ec2 run-instances \
        --count 1 \
        --image-id ami-00000000 \
        --instance-type m3.medium \
        --key-name my-keypair \
        --security-group-ids sg-00000000 \
        --user-data file://my-user-data

IRONdb instances can take up to 5 minutes to become available. Once you have launched your instance, you can find a log of the initial setup at `/root/irondb-setup.log`. Depending on the speed of your instance, setup may still be in progress when you first log in. Check the setup log for a `SETUP COMPLETE` message.

### Cluster Configuration

Additional configuration is required for clusters of more than one IRONdb node. This involves describing the topology of the cluster, including the addresses and UUIDs of the participating nodes, as well as the desired number of write copies for stored data. 

** The above setup script configures a single, standalone instance. If you have already been using such an instance, configuring it to be part of a cluster will cause your existing stored data to become unavailable. It is therefore preferable to complete cluster setup prior to ingesting any metric data into IRONdb. **

To configure a multi-node cluster, follow these steps.

#### Determine Write Copies

The number of write copies determines the number of nodes that can be unavailable before metric data become inaccessible. A cluster with N write copies can survive N-1 node failures before data become inaccessible. Clusters of up to 6 nodes should have a minimum of 2 write copies. Clusters of 6 or more nodes should have a minimum of 3 write copies, up to a maximum of 10.

#### Create Topology Layout

The topology layout describes the particular nodes that are part of the cluster as well as aspects of operation for the cluster as a whole, such as the number of write copies. The layout file is not read directly by IRONdb, rather it is used to create a canonical topology representation that will be referenced by the IRONdb config.

Since the 0.6 beta release, a helper script exists for creating the topology: `/opt/circonus/bin/topo-helper`:

    Usage: /opt/circonus/bin/topo-helper [-h] -a <start address> -i <uuid,uuid,...> -w <write copies>
      -a <start address> : Starting IP address (inclusive)
      -i <uuid,uuid,...> : List of node UUIDs
      -w <write copies>  : Number of write copies
      -h                 : Show usage summary

This will create a temporary config, which you can edit afterward, if needed, before importing. It assumes that the nodes will be addressed sequentially from the starting IP address. If this is not the case in your cluster, you can edit the IPs in the generated config before importing.

For example, in a cluster of 3 nodes, which have all been set up using `setup-irondb`, where we want 2 write copies:

    /opt/circonus/bin/topo-helper \
        -a 192.168.1.11 \
        -w 2 \
        -i '7dffe44b-47c6-43e1-db6f-dc3094b793a8,
           964f7a5a-6aa5-4123-c07c-8e1a4fdb8870,
           c85237f1-b6d7-cf98-bfef-d2a77b7e0181'

The resulting temporary config looks like this:

    <nodes write_copies="2">
      <node id="7dffe44b-47c6-43e1-db6f-dc3094b793a8"
            address="192.168.1.11"
            apiport="8112"
            port="8112"
            weight="170"/>
      <node id="964f7a5a-6aa5-4123-c07c-8e1a4fdb8870"
            address="192.168.1.12"
            apiport="8112"
            port="8112"
            weight="170"/>
      <node id="c85237f1-b6d7-cf98-bfef-d2a77b7e0181"
            address="192.168.1.13"
            apiport="8112"
            port="8112"
            weight="170"/>
    </nodes>

** There are a few important considerations for IRONdb cluster topologies: **
 * The values of `id`, `port`, and `weight`, as well as the ordering of the `<node>` stanzas are used in calculating a unique hash that identifies the topology to the system. Changing any of these on a previously configured node will invalidate the topology and cause the node to refuse to start.
 * The node address may be changed at any time without affecting the validity of the topology.
 * If a node fails, its replacement should keep the same UUID, but it can have a different IP address.

The temporary config is written out to `/tmp/topology.tmp`. You may edit this file if needed, such as to configure a split cluster (see below.)

When you are satisfied that it looks the way you want, copy this file to `/opt/circonus/etc/topology` on each node, then proceed to the [Import Topology](#import-topology) step.

##### Split Clusters

One additional configuration dimension is possible for IRONdb clusters. A cluster may be divided into two "sides", with the guarantee that at least one copy of each stored metric exists on each side of the cluster. This allows for cluster distribution across typical failure domains such as network switches, rack cabinets or physical locations.

Split-cluster configuration is subject to the following restrictions:

 * Only 2 sides are permitted.
 * An active, non-split cluster cannot be converted into a split cluster as this would change the existing topology, which is not permitted.
 * Both sides must be specified, and non-empty (in other words, it is an error to configure a split cluster with all hosts on one side only.)

To configure a sided topology, edit the temporary topology created in the previous step, adding the `side` attribute to each `<node>`, with a value of either `a` or `b`. The above sample config with sides configured might look like this:

    <nodes write_copies="2">
      <node id="7dffe44b-47c6-43e1-db6f-dc3094b793a8"
            address="192.168.1.11"
            apiport="8112"
            port="8112"
            side="a"
            weight="170"/>
      <node id="964f7a5a-6aa5-4123-c07c-8e1a4fdb8870"
            address="192.168.1.12"
            apiport="8112"
            port="8112"
            side="a"
            weight="170"/>
      <node id="c85237f1-b6d7-cf98-bfef-d2a77b7e0181"
            address="192.168.1.13"
            apiport="8112"
            port="8112"
            side="b"
            weight="170"/>
    </nodes>

#### Import Topology

This step calculates a hash of certain attributes of the topology, creating a unique "fingerprint" that identifies this specific topology. It is this hash that IRONdb uses to load the cluster topology at startup. Import the desired topology with the following command:

    /opt/circonus/bin/snowthimport \
      -c /opt/circonus/etc/irondb.conf \
      -f /opt/circonus/etc/topology

If successful, the output of the command is `compiling to <long-hash-string>`.

Next, update `/opt/circonus/etc/irondb.conf` and locate the `topology` section, typically near the end of the file. Set the value of the topology's `active` attribute to the hash reported by `snowthimport`. It should look something like this:

    <topology path="/opt/circonus/etc/irondb-topo"
              active="742097e543a5fb8754667a79b9b2dc59e266593974fb2d4288b03e48a4cbcff2"
              next=""
              redo="/irondb/redo/{node}"
    />

Save the file and restart IRONdb:
* (OmniOS) `/usr/sbin/svcadm restart irondb`
* (EL7, Ubuntu) `/bin/systemctl restart circonus-irondb`

Repeat the import process on each cluster node.

#### Verify Cluster Communication

Once all nodes have the cluster topology imported and have been restarted, verify that the nodes are communicating with one another by viewing the Replication Latency tab of the [IRONdb Operations Dashboard](operations.md#operations-dashboard) on any node. You should see all of the cluster nodes listed by their IP address and port, and there should be a latency meter for each of the other cluster peers listed within each node's box.

The node currently being viewed is always listed in blue, with the other nodes listed in either green, yellow, or red, depending on when the current node last received a gossip message from that node. If a node is listed in black, then no gossip message has been received from that node since the current node started. Ensure that the nodes can communicate with each other via port 8112 over both TCP and UDP. See the [Replication Latency tab](operations.md#replication-latency-tab) documentation for details on the information visible in this tab.


## Updating

An installed node may be updated to the latest available version of IRONdb by following these steps:

OmniOS:
1. `/usr/bin/pkg update platform/irondb`
1. `/usr/sbin/svcadm restart irondb`

EL7:
1. `/usr/bin/yum update circonus-platform-irondb`
1. `/bin/systemctl restart circonus-irondb`

Ubuntu 16.04:

We have a helper package on Ubuntu that works around issues with dependency
resolution, since IRONdb is very specific about the versions of dependent
Circonus packages, and apt-get is unable to cope with them. The helper package
must be upgraded first, i.e., it cannot be upgraded in the same transaction
as the main package.
1. `/usr/bin/apt-get install circonus-platform-irondb-apt-policy`
1. `/usr/bin/apt-get install circonus-platform-irondb`
1. `/bin/systemctl restart circonus-irondb`

In a cluster of IRONdb nodes, service restarts should be staggered so as not to jeopardize availability of metric data. An interval of 30 seconds between node restarts is considered safe.

<!-- The scripts below are for tracking the success of a marketing campaign directing people to this page and are not a part of the actual installation documentation. -->

<script type="text/javascript">var it=document.createElement("img");var u="http://engine.adzerk.net/e/22/132247/e.gif";var t=new Date().getTime();var ut=u+"?_="+t;it.src=ut;it.border=0;</script>

<!-- Twitter universal website tag code -->
<script>
!function(e,t,n,s,u,a){e.twq||(s=e.twq=function(){s.exe?s.exe.apply(s,arguments):s.queue.push(arguments);
},s.version='1.1',s.queue=[],u=t.createElement(n),u.async=!0,u.src='//static.ads-twitter.com/uwt.js',
a=t.getElementsByTagName(n)[0],a.parentNode.insertBefore(u,a))}(window,document,'script');
// Insert Twitter Pixel ID and Standard Event data below
twq('init','nv0h5');
twq('track','PageView');
</script>
<!-- End Twitter universal website tag code -->
