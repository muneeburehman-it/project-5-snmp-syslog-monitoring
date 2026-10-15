# Project 5 — SNMP & Syslog Monitoring

## Project overview
This project is a small GNS3 lab demonstrating two basic network-monitoring methods:

- **SNMP polling:** an Ubuntu host queries a Cisco IOS router for device information.
- **Remote syslog:** the router sends event messages to an Ubuntu host for centralized logging.

**Project status:** The SNMP query and receipt of syslog test messages were verified during the lab. Full persistence and recovery after recreating the Ubuntu container were not verified. This repository documents the tested results and example configurations; it does not claim a fully production-ready monitoring system.

## Lab topology

| Device | Interface | Address | Role |
|---|---|---|---|
| R-MONITOR | Gi0/0 | `192.168.50.1/24` | Cisco router on the monitoring LAN |
| Ubuntu-Monitor | eth0 | `192.168.50.10/24` | SNMP client and syslog receiver |
| R-MONITOR | Gi0/1 | DHCP (observed `192.168.255.134` during setup) | GNS3 NAT connection used during setup |

The monitoring subnet is `192.168.50.0/24`.

## How the project works

1. Ubuntu sends an SNMP request to the router at `192.168.50.1`.
2. The router responds with the requested management information.
3. The router sends configured log messages to Ubuntu at `192.168.50.10`.
4. Ubuntu's syslog service receives the messages and writes them to `/var/log/syslog`.

Common default ports:
- SNMP queries: UDP 161
- Syslog over UDP: UDP 514

## Cisco IOS configuration example

```cisco
snmp-server community public RO
logging host 192.168.50.10
logging trap informational
service timestamps log datetime msec
```

The community string `public` is included only to document the isolated lab. Do **not** use this community string on a production network.

## Ubuntu SNMP configuration example

Example entries for `/etc/snmp/snmpd.conf`:

```conf
rocommunity public 192.168.50.0/24
rocommunity6 public default -V systemonly
agentAddress udp:192.168.50.10:161
```

The `snmpd` daemon allows other systems to query the Ubuntu host itself. It is not required merely to run `snmpget` from Ubuntu to query the Cisco router; the SNMP client tools are sufficient for that test.

## Ubuntu remote syslog configuration example

Example entries for `/etc/rsyslog.conf` to receive syslog over UDP:

```conf
module(load="imudp")
input(type="imudp" port="514")
```

After changing service configuration, validate the syntax where supported and restart the relevant service. Exact commands can vary by distribution and container image.

## Validation performed

### SNMP query

Command run from Ubuntu:

```bash
snmpget -v2c -c public 192.168.50.1 1.3.6.1.2.1.1.5.0
```

Observed result:

```text
iso.3.6.1.2.1.1.5.0 = STRING: "Router.localdomain"
```

This confirms that the Ubuntu host successfully queried the router's system name using SNMPv2c during the test.

### Syslog receipt

Messages containing `PROJECT5-SYSLOG-TEST` and `PROJECT5-AFTER-RESTART` were observed in Ubuntu's `/var/log/syslog` during testing.

## What this lab demonstrates

- Basic SNMP polling from a Linux host to a Cisco IOS router
- Basic remote syslog forwarding and log inspection
- IP connectivity and service troubleshooting in GNS3
- The difference between polling a device and receiving event messages

## Limitations and next steps

- Full persistence and recovery after recreating the Ubuntu container were not verified.
- Restoring the complete saved backup archive was not confirmed.
- A clean end-to-end retest after the container reset was not verified.
- A native GNS3 topology export and screenshots are not included in this documentation bundle.
- No dashboard, alerting workflow, or long-term log-retention design was implemented.

## Production security recommendations

- Prefer SNMPv3 with authentication and privacy instead of SNMPv2c community strings.
- Restrict monitoring access with ACLs/firewall rules to authorized management hosts.
- Use unique credentials and avoid default community strings such as `public`.
- Protect and back up monitoring configuration and logs.
- Confirm time synchronization so device and syslog timestamps can be correlated.

## Repository contents

- `docs/current-status.md` — tested results and known limitations
- `docs/troubleshooting.md` — issues and troubleshooting notes
- `configs/router-monitoring.txt` — Cisco IOS example commands
- `configs/snmpd.conf.example` — Ubuntu SNMP daemon example
- `configs/rsyslog-udp.conf.example` — Ubuntu UDP syslog receiver example
