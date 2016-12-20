# IRONdb Installation

## System Requirements

IRONdb requires [OmniOS](https://omnios.omniti.com/), version r151014. Hardware requirements will necessarily vary depending upon system scale and cluster size. Please contact [sales@circonus.com](/mailto:sales@circonus.com) with questions regarding hardware sizing. Circonus recommends the following minimum hardware specification for the single-node, free, 25K-metrics option:

* 1 CPU
* 4 GB RAM
* SSD-based storage, 20 GB available space

## Installation Steps

Follow these steps to get IRONdb installed on your system. If you are using one of our pre-built Amazon EC2 images, these steps are already done for you, and your free-25K instance will be configured automatically on first boot. Please refer to [EC2 installation](#ec2-installation) below.

System commands must be run as a privileged user, such as `root`, or via `sudo`.

1. Add the Circonus IPS package publisher:

        /usr/bin/pkg set-publisher -g http://updates.circonus.net/omnios/r151014/ circonus
1. Install the package:

        /usr/bin/pkg install pkg:/platform/irondb
1. Prepare site-specific information for setup. These values may be set via shell environment variables, or as arguments to the setup script. The environment variables are listed below.
   * ##### IRONDB\_NODE\_UUID

     *\(required\)* The ID of the current node, which must be unique within a given cluster. You may use the installed `uuidgen` command or generate a UUID with an external tool or website.
   * ##### IRONDB\_NODE\_ADDR

     *\(required\)* The IPv4 address of the current node, e.g., "192.168.1.100".
   * ##### IRONDB\_CHECK\_UUID

     *\(required\)* Check ID for Graphite metric ingestion, which must be the same on all cluster nodes. You may use the installed uuidgen command or generate a UUID with an external tool or website.
   * ##### IRONDB\_CHECK\_NAME

     *\(required\)* The string that will identify Graphite-compatible metrics stored in the check identified by `IRONDB_CHECK_UUID`. For example, if you submit a metric named "my.metric.1", and the check is named "test", the resulting metric name in IRONdb will be "graphite.test.my.metric.1".
   * ##### IRONDB\_CRASH\_REPORTING

     *\(optional\)* Control enablement of automated crash reporting. Default is "on". IRONdb utilizes sophisticated crash tracing technology to help diagnose errors. Enabling crash reporting requires that the system be able to connect out to the Circonus reporting endpoint: https://circonus.sp.backtrace.io:6098 . If your site's network policy forbids this type of outbound connectivity, set the value to "off".
   * ##### IRONDB\_ZPOOL

     *\(optional\)* The name of the zpool that should be used for IRONdb storage. If not specified, and there are multiple zpools in the system, setup chooses the pool with the most available space.
1. Run the setup script. All required options must be present, either as environment variables, or via command-line arguments. A mix of environment variables and arguments is permitted, but environment variables take precedence over command-line arguments. Use the `-h` option to view a usage summary:

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
1. Obtain your license information from your Circonus account profile: https://YOURACCOUNT.circonus.com/profile

   If you do not have an IRONdb license, click the `+` to the right of the Licenses section of the account profile page to add a new license.

   If you do not have a Circonus account, sign up for free at https://login.circonus.com/signup .
1. Add the `<license>` stanza from your chosen IRONdb license to the file `/opt/circonus/etc/licenses.conf` on your IRONdb instance, within the enclosing `<licenses>` tags. It should look something like this:

        <licenses>
        <license id="100" sig="(cryptographic signature)">
          <graphite>true</graphite>
          <max_streams>25000</max_streams>
          <company>MyCompany</company>
        </license>
        </licenses>
1. Restart the IRONdb service:

        /usr/sbin/svcadm restart irondb

## EC2 Installation

Circonus makes available EC2 AMIs that come preinstalled with IRONdb. The first time an instance of the AMI boots, the setup script runs and configures a standalone instance. The AMI may be used on any instance type supported by OmniOS \(currently only PV instances are supported\), and the minimum recommended instance type is `m3.medium`.

Setup expects the required options to be provided as instance user-data. When launching your instance, add the necessary options in environment-variable format:

        IRONDB_NODE_UUID="123e4567-e89b-12d3-a456-789000000000"
        IRONDB_CHECK_NAME="test"
        IRONDB_CHECK_UUID="987f6543-e21d-98c7-b654-a32100000000"
        IRONDB_CRASH_REPORTING="on"

The setup process will detect the local IP address of the instance at boot, so it is not necessary to specify `IRONDB_NODE_ADDR` in the user-data \(it will be ignored even if present.\)

Setup also expects to configure a zpool on the secondary EBS volume that is specified by the AMI. You do not need to specify `IRONDB_ZPOOL` in the user-data.

IRONdb instances can take up to 5 minutes to become available. Once you have launched your instance, you can find a log of the initial setup at `/root/irondb-setup.log`. Depending on the speed of your instance, setup may still be in progress when you first log in. Check the setup log for a `SETUP COMPLETE` message.
