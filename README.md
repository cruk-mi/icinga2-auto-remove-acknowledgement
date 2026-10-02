# icinga2-auto-remove-acknowledgement
Removing acknowledgement on output text change

An Icinga 2 event-command helper that automatically removes a **service acknowledgement when the service remains in a non-OK state but its plugin output changes**.

This is useful for services where changed output represents a new or materially different problem that should be reviewed and acknowledged again. The supplied service example uses `volatile = true` so that the event command runs for each non-OK check result.

##Use case

You acknowledge a CRITICAL service because you are dealing with it. The service remains CRITICAL, but later the plugin output changes because the underlying problem has changed. Normally the acknowledgement remains. This plugin automatically removes the acknowledgement so the new condition can be reviewed and notified again.

##Topics

icinga2
icinga
icinga-plugin
monitoring
nagios
acknowledgement
event-command
monitoring-plugin

## Behaviour

1. When an acknowledged, non-OK service is first observed, the current plugin output is stored.
2. On later checks:
   - unchanged output: keep the acknowledgement;
   - changed output: call the Icinga 2 REST API to remove the acknowledgement;
   - OK or no longer acknowledged: remove the local tracking file.
3. If the API call fails, the script returns exit code 3 and retains the tracking file.

## Important limitations

- Services only. The API request uses `type: Service`.
- Comparison is based on the complete plugin output string.
- Each execution endpoint needs a writable state directory and API connectivity.
- The event command must be invoked on every relevant non-OK result. The example uses a volatile service.
- Review this logic in a test environment before production use. Automatically removing acknowledgements can resume notifications.

## Requirements

- Icinga 2 with the API feature enabled
- Bash
- `curl`, `python3`, `stat`, `date`, and standard core utilities
- An Icinga API user permitted to run `actions/remove-acknowledgement`
- A trusted CA certificate for the Icinga API endpoint

## Installation

```bash
sudo install -m 0755 auto_remove_ack /usr/lib64/nagios/plugins/auto_remove_ack
sudo install -d -o icinga -g icinga -m 0750 /var/lib/icinga2/output_tracking
```

Adjust the plugin path and service account for your distribution. Debian-family systems commonly use a different plugin directory.

Configure credentials securely. The script accepts these environment variables:

```bash
APIUSER='auto-remove-ack'
APIPASS='replace-with-a-secret'
APIURL='https://icinga.example.org:5665'
LOGFILE='/var/log/icinga2/auto_remove_ack.log'
STATE_DIR='/var/lib/icinga2/output_tracking'
```

Do not commit real credentials. For production, use an Icinga-compatible secret-management method or a root-owned wrapper/configuration readable only by the execution account.

## Icinga configuration

Copy and adapt `examples/event-command.conf` and `examples/service.conf`.

Validate and reload:

```bash
sudo icinga2 daemon -C
sudo systemctl reload icinga2
```

## API user

An example is provided in `examples/api-user.conf`. Restrict permissions and object filters to the smallest practical scope. The Icinga API uses HTTPS and supports API users for authentication.

## sudo

The example event command calls the plugin directly. If your deployment requires `sudo`, use a narrowly scoped sudoers entry such as `examples/sudoers-auto-remove-ack`; do not grant unrestricted sudo access.

## Testing

1. Apply the event command to a non-production service.
2. Force a non-OK result and acknowledge it.
3. Repeat the same output and confirm the acknowledgement remains.
4. Change the plugin output while retaining the non-OK state.
5. Confirm the acknowledgement is removed and review the log.
6. Test an API failure and confirm the state file is retained.

## Security notes

- The original local version used `curl -k`; this public version deliberately does not disable TLS verification.
- Host and service names are sanitized before constructing the state filename.
- JSON is generated with Python to avoid malformed JSON when names contain quotes or special characters.
- API failures are not treated as successful removals.

See `SECURITY.md` for reporting security issues.

## Licence
GPL v3


## Disclaimer

This is a community project and is not an official Icinga component. Test thoroughly and ensure the behaviour matches your notification and incident-management policy.
