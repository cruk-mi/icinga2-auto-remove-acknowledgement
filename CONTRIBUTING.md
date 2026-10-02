# Contributing

Thank you for helping improve this project.

1. Open an issue describing the problem or proposed enhancement.
2. Do not include passwords, hostnames, internal URLs, or production logs containing sensitive data.
3. Keep changes focused and include a clear test procedure.
4. Run `bash -n auto_remove_ack` and, where available, `shellcheck auto_remove_ack`.
5. Validate example Icinga configuration with `icinga2 daemon -C` in a test environment.
