check-mk plugin for DataCore SSV
================================


About
-----

This check-mk package retrieves the monitor states of *DataCore
SANsymphony-V* groups using SNMP and creates corresponding services.


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

Before *ssv_monitors* is able to discover the DataCore monitors you need
to install and configure the *SNMP Service* service of the MS Windows OS.
Additional the *DataCore SNMP Agent* service needs to be started and the
start type should be changed to `Automatic`. Please consider to take a
look at DataCore's *SSV WebHelp*[2] about the SNMP setup.

[2] http://www.datacore.com/SSV-WebHelp/SNMP_Support2.htm

Once the SNMP setup is finished *check-mk* is able to discover serveral new
services at your DataCore servers.

```console
$ check_mk -II dcore1.fqdn
hp_proliant       1 new checks
hp_proliant_cpu   2 new checks
hp_proliant_da_cntlr 1 new checks
hp_proliant_da_phydrv 8 new checks
hp_proliant_fans  6 new checks
hp_proliant_mem   12 new checks
hp_proliant_psu   2 new checks
hp_proliant_temp  44 new checks
hp_sts_drvbox     2 new checks
hr_cpu            1 new checks
hr_fs             1 new checks
hr_mem            1 new checks
if                16 new checks
snmp_info         1 new checks
snmp_uptime       1 new checks
ssv_monitors      249 new checks
```

Due to DataCore's SNMP implementation any DataCore server within a
DataCore group will return the same monitor items. We recommend to use
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

Some of the monitor names already contain the DataCore server name *dcore1*
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
case sensitive). If your group contains more than two servers you need to
extend the regex or ruleset accordingly.
