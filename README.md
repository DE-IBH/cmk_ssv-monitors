DataCore SANsymphony-V SNMP Monitors
====================================


About
-----

This check retrieves the monitor states of *DataCore SANsymphony-V*
installations using SNMP.


Install
-------

Download the provided *check-mk package file* `ssv_monitors-x.y.mkp` and
install it using *check-mk*'s package manager[1].

[1] https://mathias-kettner.de/checkmk_packaging.html#H1:Installation,%20Update%20and%20Removal

```console
# check_mk -P install ssv_monitors-x.y.mkp
```

Setup
-----

Before *ssv_monitors* is able to discover the DataCore Monitors you need
to install and configure the *SNMP Service* service of the MS Windows OS.
Additional the *DataCore SNMP Agent* service needs to be started and the
start type should be changed to `Automatic`. Please consider to take a
look at DataCore's SSV WebHelp[2] about the SNMP setup.

[2] http://www.datacore.com/SSV-WebHelp/SNMP_Support2.htm

Once the SNMP setup is finished check_mk is able to discover serveral new
services at your DataCore Servers.


Due to DataCore's SNMP implementation any DataCore Server within a
DataCore Cluster will return the same monitor items. We recommend to use
a naming scheme for your physical disks, disk pools, server ports etc.
to be able to use *check-mk*'s rule engine to filter them.

```console
$ check_mk -nv dcore1.fqdn
Check_mk version 1.2.4p5
CPU utilization      OK - 13.4% utilization at 24 CPUs
SSV DiskPool dcore1-sas OK - status is Healthy: Running
SSV DiskPool dcore1-sata OK - status is Healthy: Running
SSV DiskPoolFree dcore1-sas OK - status is Healthy: Available space > 5%
SSV DiskPoolFree dcore1-sata CRIT - status is Critical: Available space <= 5%
SSV LogicalDisk xenpool_01 on dcore1 OK - status is Healthy: Online
SSV LogicalDiskAccess xenpool_01 on dcore1 OK - status is Healthy: Read/Write
SSV LogicalDiskData xenpool_01 on dcore1 OK - status is Healthy: Up to date
SSV LogicalDiskFailure xenpool_01 on dcore1 OK - status is Healthy: No failure
SSV PhysicalDisk HP 3PAR DCORE1-01 OK - status is Healthy: On-line
SSV ServerFcPort FC-dcore1 2p1 OK - status is Healthy: Loop/Link up
SSV ServerMSiScsiInitiator Microsoft iSCSI Initiator dcore1 OK - status is Healthy: Present
SSV ServerMachine dcore1 OK - status is Healthy: Running
SSV ServerMachineLicense dcore1 OK - status is Healthy: License expires in more than 14 days
SSV ServerMachinePower dcore1 OK - status is Healthy: AC line power available
SSV ServerPortLinkError FC-DCORE1 2p1 WARN - status is Warning: 18 link errors
SSV ServeriScsiPort LAN-DCORE1-1 OK - status is Healthy: Ready
SSV StreamLogicalDisk xenppol_01 on dcore1 OK - status is Healthy: Continuous data protection enabled
SSV vDiskAccess xenpool_01 OK - status is Healthy: Read/Write
SSV vDiskData xenpool_01 OK - status is Healthy: Up to date
```

Some of the monitor names already contain the DataCore Server name *dcore1*
(_LogicalDisk*_, _ServerMachine_, _StreamLogicalDisk_). Disk pools and
server ports need to be renamed to contain the hostname. To filter the
non-local monitor items you could use *check-mk* rules:

```python
ignored_services = [
  ( [], ['dcore1.fqdn'], ['.+\\b(dcore|DCORE)2\\b'] ),
  ( [], ['dcore2.fqdn'], ['.+\\b(dcore|DCORE)1\\b'] ),
] + ignored_services
```

Each rule matches on the hostname. On *dcore1.fdqn* the rule filters any
service containing *dcore2* using a regex (`\b` is a word boundary, regex are
case sensitive). If your cluster contains more than two service you need
to extend the regex or ruleset accordingly.
