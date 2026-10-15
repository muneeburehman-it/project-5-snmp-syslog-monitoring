# Troubleshooting Notes

This file records the main issues encountered during the lab. It is a summary, not a complete command-by-command transcript.

## 1. SNMP query did not work at first

Check the following in order:

1. Confirm the router and Ubuntu host have addresses in the same intended subnet.
2. Test basic IP connectivity between the hosts.
3. Confirm the Cisco SNMP community and read-only access configuration.
4. Confirm that the SNMP client is querying the correct router address and OID.
5. Check ACLs and firewall rules for UDP 161.
6. Confirm the router's SNMP configuration with appropriate IOS `show` commands.

Successful test recorded in this lab:

```bash
snmpget -v2c -c public 192.168.50.1 1.3.6.1.2.1.1.5.0
```

Expected recorded response:

```text
iso.3.6.1.2.1.1.5.0 = STRING: "Router.localdomain"
```

## 2. Syslog messages were not immediately visible

Check:

1. The router's logging destination is `192.168.50.10`.
2. The Ubuntu host has the expected address and can receive packets.
3. The rsyslog UDP input is enabled on port 514.
4. Host/container firewall rules allow UDP 514.
5. The rsyslog service is running and `/var/log/syslog` is being written.
6. Generate a test log message and search the log file.

Recorded markers that were seen in `/var/log/syslog`:

- `PROJECT5-SYSLOG-TEST`
- `PROJECT5-AFTER-RESTART`

## 3. Container reset or recreation

During troubleshooting, the Ubuntu container was recreated/reset. Some network settings and packages had to be restored, and the complete restore process was not verified end-to-end.

For repeatable labs:

- Keep configuration copies outside the ephemeral container.
- Save the GNS3 project and relevant device configurations.
- Record IP addressing and package requirements.
- After a reset, confirm interfaces and addresses, installed packages, service state, and connectivity before retesting SNMP and syslog.
- Do not mark recovery as successful until the complete test has passed again.

## Security reminder

The example uses SNMPv2c and the community string `public` for a lab demonstration only. Production deployments should use SNMPv3 where possible, restrict management traffic, and avoid default or shared community strings.
