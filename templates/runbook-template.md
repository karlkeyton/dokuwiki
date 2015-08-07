{nocamel}

h1. <name-of system ie: FRPP-ADE>

The following run-book describes the specifics relating to the configuration and runtime of the <name-of system ie: FRPP-ADE> hosts in our environment. This guide should be utilized by Production Systems Engineering and all other teams who need to support this platform as a general resource for how to maintain and operate these hosts. Additions of any relevant data are strongly encouraged, and this page should be audited regularly to ensure correctness.

{toc}

h2. Points of Contact

The follow section describes the points of contact for this service from various business contexts.

{table-plus:width=900}


|| Context || Team || Confluence Space || [PagerDuty] Service || Engineer (optional) ||
| Operational | Production [SysEng] | [TechOps|https://wiki.rubiconproject.com/pages/viewpage.action?pageId=14551360] | [TechOps Service|https://rubiconproject.pagerduty.com/services/PLJZ36X] | Consult Pagerduty for on-call |
| Development | Data Pipeline | [Data Pipeline|https://wiki.rubiconproject.com/display/Tech/Data+Pipeline] | [Data Pipeline Service|https://rubiconproject.pagerduty.com/services/P20S25M] | Consult Pagerduty for on-call |
| Quality Assurance | QA | [QA|https://wiki.rubiconproject.com/display/Tech/QA] | [QA Service|https://rubiconproject.pagerduty.com/services/PKU7OPW] | Consult Pagerduty for on-call |
| Project Management | Project Management | [Tech:Project Management] | [Project Management|https://rubiconproject.pagerduty.com/services/PM9F0NG] | Consult Pagerduty for on-call |
{table-plus}

h2. Hardware

The following section describes the hardware considerations for this host role.

h3. Hosts per datacenter

Below is the intended host count per datacenter.
{table-plus:width=900}


|| FRA1 || AMS2 || IAD1 || LAS1 || LAS2 || LAB1 || LAX2 || SJC1 || NRT1 ||
| 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
{table-plus}

h3. Hardware profile

Below is a description of the intended hardware configuration for this host role.

{table-plus:width=900}


|| Name || Hardware Description || Notes ||
| STANDARD Infra HW Profile | [DL360pG8], 32GB RAM, [SmartArrayP420i], 2 x EH0146FARWD (146GB 15K SAS, RAID1) | ... |
{table-plus}

h3. Measures taken for host-level redundancy

h3. Hardware-centric Graphite Dashboard

h3. Known Issues

*NONE* at present.

h2. Network

The following section describes the network considerations for this host role.

h3. Host interfaces

Below is a description of the intended network interface configurations for these hosts.

h3. External requirements

Ext Requirements

h3. Load Balancing requirements

*NONE* at present.

h3. Special considerations

*NONE* at present. This service is relatively low volume.

h3. Known Issues

*NONE* at present.

h2. Software

The following section describes the software considerations for this host role.

h3. Puppet classes

Below is a per-class description of the specifics behind the Puppet classes applied to these hosts.

h3. In-house packages

Below are the specifics for each internally-developed RPM package installed for this host role.

{table-plus:width=900}


|| Package Name || Current Production Version || Maintainer || Notes ||
| None. | ... | ... | ... |
{table-plus}

h3. External packages

Below are the specifics for each externally-developed RPM package installed for this host role.

{table-plus:width=900}


|| Package Name || Current Production Version || Maintainer || Notes ||
{table-plus}

h3. Release Process (if not standard)

*NONE* at present.

h3. Known Issues

*NONE* at present.

h2. Configurations

Below are the specifics for each configuration file utilized by the core software installed for this host role.

h2. Operational considerations

Below are the specifics for the operational considerations for this host role, including crontabs, system services, ports utilized, host-to-host communications, and log files.

h3. Crontabs

Below are the specifics for each crontab installed specifically for this host role.

h3. Services

Below are the specifics for each system service configured distinctly for this host role.

{table-plus:width=900}


|| Service || Uses SysV || Sysconfigs || Notes ||
{table-plus}

h3. Local TCP/UDP ports

Below are the specifics for each local TCP or UDP port utilized by services for this host role.

{table-plus:width=900}


|| Process || Interfaces || Type || Port || Notes ||
{table-plus}

h3. Communication points

Below are the specifics for the known communication points with other hosts for this host role.

{table-plus:width=900}


|| Process || Interface || Destination Host || Destination Port || Notes ||
{table-plus}

h3. Log files

Below are the specifics for the log files available for this host role.

h4. /var/log/somelog

*NONE* at present.

h3. Troubleshooting steps

*NONE* at present.

h3. Known issues

*NONE* at present.

h2. Monitoring

The following section describes the monitoring considerations for this host role.

h3. Current Service Checks

Below is a per-check description of the specifics behind the alerts applied to these hosts.

{table-plus:width=900}


|| Name || Platform || Check details || Tech Summary || Biz Summary || Escalation || SLA || Notes ||
{table-plus}

h3. Resolution steps

Below is a per-check description of corrective action to be taken for the alerts applied to these hosts. *NOTE:* the below steps utilize fopp-emx0000.fra1.fanops.net as an example. Replace as needed for the alert received.

h4. <Alert in table above>

h3. List of related Graphite Dashboards

h3. Known Issues
