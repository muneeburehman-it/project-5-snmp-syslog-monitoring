# Current Status and Test Evidence

## Summary

The lab connected a Cisco IOS router and an Ubuntu monitoring host on `192.168.50.0/24`.

| Test | Recorded result | Status |
|---|---|---|
| SNMP query from Ubuntu to router | `snmpget` returned `Router.localdomain` | Verified during lab |
| Remote syslog receipt | Test messages appeared in `/var/log/syslog` | Verified during lab |
| Full recovery after Ubuntu container recreation | Not fully retested and verified | Unverified |
| Dashboard or automated alerts | Not implemented | Not implemented |
| Native GNS3 project and screenshots in this bundle | Not included | Not included |

## Addressing

- Router `Gi0/0`: `192.168.50.1/24`
- Ubuntu monitoring host `eth0`: `192.168.50.10/24`
- Monitoring subnet: `192.168.50.0/24`
- Router `Gi0/1`: DHCP address observed as `192.168.255.134` during setup

## SNMP test

Command:

```bash
snmpget -v2c -c public 192.168.50.1 1.3.6.1.2.1.1.5.0
```

Observed output:

```text
iso.3.6.1.2.1.1.5.0 = STRING: "Router.localdomain"
```

This confirms a successful SNMP query at the time it was run. It does not by itself prove that every OID, interface counter, or long-term monitoring feature was tested.

## Syslog test

The following markers were observed in `/var/log/syslog`:

- `PROJECT5-SYSLOG-TEST`
- `PROJECT5-AFTER-RESTART`

These observations confirm receipt of those test messages during the lab. They do not establish that logging survives every container restart or recreation.

## Honest project description for a portfolio

> Built a basic Cisco-to-Ubuntu monitoring lab in GNS3. Configured SNMP polling to retrieve the router's system name and configured remote syslog forwarding. Verified the SNMP response and observed test log messages on the Ubuntu host. Documented the configuration and identified persistence/recovery as remaining validation work.

## Important note

Configuration files in `configs/` are examples based on the lab notes. Review and adapt them before applying them to any system. They are not a substitute for the actual device configuration backup.
