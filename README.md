# DDN External Checkers

## About the Project

DDN External Checkers is a collection of elementary checks aimed at complementing the monitoring of a DDN SFA-based storage infrastructure.

In a DDN storage infrastructure environment, main supervision is expected to be achieved through the use of the SFA OS API, but a couple of items which are not available through the SFA OS API need to be monitored separately in order to:
- Provide insight on the storage infrastructure status
- Prevent forthcoming failures

DDN External Checkers are divided into two distinct, complementary modules:
- DDN SFA Checkers: Monitoring of DDN SFA controllers
- DDN EXAScaler Checkers: Monitoring of DDN Object Storage Servers (OSS)

DDN External Checkers produce ASCII files as an output. In an integrated monitoring platform, these files should ideally serve as input data for the main monitoring solution (Nagios, Icinga).

#### DDN EXAScaler Checkers

The DDN EXAScaler Checkers performs the following checks:

| Check             | Purpose
|-------------------|----------------------------------------|
| Collectd          | collectd systemd Unit Status
| Connection to MGS | Loss of Connectivity to MGS
| Degraded          | Degraded State
| Filesystems       | Local Filesystems Occupancy
| HA Resources      | High Availability Resources Status
| High CPU Load     | Occurrences of High CPU Load Warnings
| MB C3 Threshold   | MB C3 Threshold Setting
| Uptime            | Uptime

##### Collectd

* Function: `ddn-es-checker::collectd`
* Purpose: Reports `collectd` systemd unit status.
* Type: Failure Detection
* Output:
  ```
	sfa01o01: collectd: status=active
	```

##### Connection to MGS

* Function: `ddn-es-checker::connection-to-mgs`
* Purpose: Counts number of times connection to MGS was lost on the current day. Frequent disconnections from the MGS might be a symptom of a forthcoming failure.
* Type: Preventive Failure Detection
* Output:
  ```
	sfa01o01: connection-to-mgs: lost=0
	```

##### Degraded

* Function: `ddn-es-checker::degraded`
* Purpose: Reports OSS which have been put in degraded state. Degraded state is required to prevent an OSS which starts exhibiting errors from failing.
* Type: Storage Infrastructure Status Monitoring

##### Filesystems

* Function: `ddn-es-checker::filesystems`
* Purpose: Reports `/` and `/var` OSS filesystems occupancy. Full occupancy of one of the local filesystems would make the high-availability processes fail immediately.
* Type: Preventive Failure Detection
* Output:
  ```
  sfa01o01: filesystems: fs-root=72;fs-var=12
	```

##### High-Availability Resources

* Function: `ddn-es-checker::ha-resources`
* Purpose: Counts number of high-availability resources which are currently configured / disabled. All high-availability resources should be all configured whenever the OSS are in optimal condition.
* Type: Preventive Failure Detection
* Output:
  ```
	sfa01o01: ha-resources: configured=42;disabled=
  ```

##### High CPU Load

* Function: `ddn-es-checker::high-cpu-load`
* Purpose: Counts number of occurrences and spots maximum / average CPU load. Frequent high CPU load messages are often a symptom of a forthcoming failure.
* Type: Preventive Failure Detection
* Output:
  ```
	sfa01o01: high_cpu_load: avg=0;max=0;count=0
  ```

#### SFA External Checker

The SFA External Checker performs the following checks:

| Check             | Purpose
|-------------------|----------------------------------------|
| Position Status   | Position Status of Enclosures
| SCSI Events       | Occurrences of SCSI Events
| Write-Back        | Write-Back Feature Status

##### Position Status

* Function Name: `ddn-sfa-checker::position-status`
* Purpose: Reports position status of every enclosure (SAS cables connection state). Value different from success would tend to indicate that SAS cables connection scheme has not been respected.
* Type: Storage Infrastructure Status Monitoring
* Action:
  ```
	sfa01c0: position-status: 0=success,1=success,2=success,3=success,4=success,5=success,6=success,7=success
	```

##### SCSI Events

* Function Name: `ddn-sfa-checker::scsi-events`
* Purpose: Counts number of occurrences of SCSI events over the last 3 days.
  SCSI events often indicate a forthcoming failure either of a SAS cable or an IO Module.
* Type: Preventive Failure Detection
* Output:
  ```
  sfa01c0: scsi-events: 2021-10-15=0;2021-10-14=0;2021-10-13=0
  ```

##### Write-Back

* Function Name: `ddn-sfa-checker::write-back`
* Purpose: Reports storage controllers with Write-Back feature disabled (Write-Through Mode). Write-Back should not be kept disabled longer that required by a specific maintenance operation.
* Type: Storage Infrastructure Status Monitoring
* Action:
  ```
  sfa01c0: write-back: 0=true,1=true,2=true,3=true,4=true,5=true,6=true,7=true
	```

## Getting Started

#### Prerequisites

- xCAT Management Server:
  - Distributed Shell Command `xdsh`
  - DDN:SFA Specific Distributed Shell Configuration
	  ```
    $ cat /opt/xcat/share/xcat/devicetype/DDN/SFA/config
    [main]
    [xdsh]
    pre-command=NULL
    post-command=NULL

  > Note: Use of xCAT Distributed Shell `xdsh` can be easily replaced by a different distributed shell - like `pdsh` for instance - if preferred.

- Target Objects:
  - DDN SFA storage controllers
  - Lustre Object Storage Servers (OSS)

#### Installation

Clone Git repository:
```
$ git clone https://github.com/nicolas-tallet/ddn-external-checkers.git
$ export PATH="${PWD}/ddn-passive-checkers/bin:${PATH}"
```

#### Usage

```
$ ddn-external-checkers -noderange-es NODERANGE_ES -noderange-sfa NODERANGE_SFA -system SYSTEM
```

Example:
```
$ ddn-external-checkers -noderange-es "sfa01cont[01-02]" -noderange-sfa "sfa01oss[01-08]" -system "sfa01"
```