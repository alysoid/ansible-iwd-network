# [Catena](https://github.com/alysoid/catena) Ansible Role: iwd-network

Manage system wireless network via [iwd](https://man.archlinux.org/man/iwd.8.en) (iNet wireless daemon), a daemon for managing Wireless devices on Linux. iwd can work in standalone mode or in combination with comprehensive network managers like systemd-networkd. This Ansible role works well with [systemd-networkd role](https://github.com/alysoid/ansible-systemd-networkd) that uses systemd-networkd to manage system network, and [systemd-resolved role](https://github.com/alysoid/ansible-systemd-resolved) that manages system DNS resolver with systemd-resolved.

## Role default variables

| Variable                     | Default | Info                                                                                  |
| ---------------------------- | ------- | ------------------------------------------------------------------------------------- |
| `catena_iwd_conf`            | `[]`    | Manage `main.conf` configuration file. Configures system-wide settings for iwd.       |
| `catena_iwd_network`         | `[]`    | Create `/var/lib/iwd/*.open,*.psk,*.8021x` wireless configuration files for networks. |
| `catena_iwd_conf_cleanup`    | `no`    | Remove existing configuration files: `/etc/iwd/*.conf`.                               |
| `catena_iwd_network_cleanup` | `no`    | Remove existing network configuration files: `/var/lib/iwd/*.open,*.psk,*.8021x`.     |

### `catena_iwd_conf`

Manage [iwd.config](https://man.archlinux.org/man/iwd.config.5) - Configuration file for wireless daemon.

The `/etc/iwd/main.conf` configuration file configures the system-wide settings for iwd. If no main.conf is present, then default values are chosen.

```yaml
# Default
catena_iwd_conf: []

# Example
catena_iwd_conf:
  # Create `/etc/iwd/main.conf`
  - name: main
    options:
      # Define [General] section
      General:
        # Network configuration feature disabled by default
        EnableNetworkConfiguration: "false"
      # Define [Network] section
      Network:
        # Resolution method used by the system. By default is systemd
        NameResolvingService: "systemd"
      # Define [Scan] section
      Scan:
        # Prevent periodic scans for available networks while disconnected. By default is true.
        DisablePeriodicScan: "true"
```

### `catena_iwd_network`

Manage [iwd.network](https://man.archlinux.org/man/iwd.network.5) - Network configuration for wireless daemon.

iwd monitors the directory `/var/lib/iwd` for changes and updates its state accordingly. Network configuration files are based on the network's SSID and security type. The name consist of the encoding of the SSID followed by `.open`, `.psk` or `.8021x`.

```yaml
# Default
catena_iwd_network: []

# Example
catena_iwd_network:
  # Wireless network configuration files (.open, .psk or .8021x)
  # Create `/var/lib/iwd/wifi_network.psk`, wifi_network is the SSID
  - name: wifi_network.psk
    options:
      # Define [Security] section
      Security:
        # WPA-Personal password for the network.
        # Required for WPA3-Personal (SAE) or if PreSharedKey is not provided.
        Passphrase: "{{ wifi_Passphrase }}"
        # Processed 64-char hex string passphrase for the network.
        # Mandatory if Passphrase is omitted.
        PreSharedKey: "{{ wifi_PreSharedKey }}"
  # Create `/var/lib/iwd/ap_network.8021x` file, where ap_network is the SSID
  - name: ap_network.8021x
    options:
      # Define [Security] section
      Security:
        # Network Authentication Settings
        # https://man.archlinux.org/man/iwd.network.5#Network_Authentication_Settings
        EAP-Method: PWD
        EAP-Identity: "{{ ap_Email }}"
        EAP-Password: "{{ ap_Password }}"
      # Define [Settings] section
      Settings:
        # Automatically connect to this network
        AutoConnect: "true"
```

When only the Passphrase is provided, iwd will automatically append the PreSharedKey to the file at the first connect. By accessing the machine, the PreSharedKey can be retrieved reading the file `/var/lib/iwd/{SSID}.psk`. After that, only the PreSharedKey can be passed to the role in order to use the network.
