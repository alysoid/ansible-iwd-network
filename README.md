# [Catena](https://github.com/alysoid/catena) - Wireless Network Ansible Role

Manage system wireless network via [iwd](https://man.archlinux.org/man/iwd.8.en) (iNet wireless daemon), a daemon for managing Wireless devices on Linux. iwd can work in standalone mode or in combination with comprehensive network managers like systemd-networkd. This Ansible role works well with [Catena Network](https://github.com/alysoid/ansible-role-network) role that uses systemd-networkd to manage system network, and [Catena Resolve](https://github.com/alysoid/ansible-role-resolve) that manages system DNS resolver with systemd-resolved.

## Role default variables

| Variable               | Default | Info
| ---------------------- | ------- | ------------------
| `iwd_conf`             | `{}`    | Manage `main.conf` configuration file. Configures system-wide settings for iwd.
| `iwd_network`          | `{}`    | Create `/var/lib/iwd/*.open,*.psk,*.8021x` wireless configuration files for networks.
| `iwd_conf_cleanup`     | `no`    | Remove existing configuration files: `/etc/iwd/*.conf`.
| `iwd_network_cleanup`  | `no`    | Remove existing network configuration files: `/var/lib/iwd/*.open,*.psk,*.8021x`.

---

### `iwd_conf`

Manage [iwd.config](https://man.archlinux.org/man/iwd.config.5) - Configuration file for wireless daemon.

The `/etc/iwd/main.conf` configuration file configures the system-wide settings for iwd. If no main.conf is present, then default values are chosen.

```yaml
iwd_conf:
  # Create `/etc/iwd/main.conf`
  main:
    # Define [General] section
    - General:
      # The network configuration feature is disabled by default
      - EnableNetworkConfiguration: 'false'
    # Define [Network] section
    - Network:
      # Resolution method used by the system. By default is systemd
      - NameResolvingService: 'systemd'
    - Scan:
      # Prevent periodic scans for available networks while disconnected. By default is true.
      - DisablePeriodicScan: 'true'
```

### `iwd_network`

Manage [iwd.network](https://man.archlinux.org/man/iwd.network.5) - Network configuration for wireless daemon.

iwd monitors the directory `/var/lib/iwd` for changes and updates its state accordingly. Network configuration files are based on the network's SSID and security type. The name consist of the encoding of the SSID followed by `.open`, `.psk` or `.8021x`. 

```yaml
# Wireless network configuration files .open, .psk and .8021x
iwd_network:
  # Create `/var/lib/iwd/wifi_network.psk` file, where wifi_network is the SSID
  wifi_network.psk:
    # Define [Security] section
    - Security:
      # Processed 64-char hex string passphrase for the network. Mandatory if Passphrase is omitted.
      - PreSharedKey: 'qvuvsbqyseuemycpscdgufdcwzdpbxpeyyptaxrdtbabibomlieqhhgivlknvpro'
      # WPA-Personal password for the network. Required for WPA3-Personal (SAE) or if PreSharedKey is not provided.
      - Passphrase: 'wwjvochcfexiiktayfonooxs'
  # Create `/var/lib/iwd/ap_network.8021x` file, where ap_network is the SSID
  ap_network.8021x:
    # Define [Security] section
    - Security:
      # Provide EAP-PWD info to access 
      - EAP-Method: PWD
      - EAP-Identity: your_email
      - EAP-Password: your_pass
    # Define [Settings] section
    - Settings:
      # Automatically connect to this network
      - AutoConnect: 'true'
```

When only the Passphrase is provided, iwd will automatically append the PreSharedKey to the file at the first connect. By accessing the machine, the PreSharedKey can be retrieved reading the file `/var/lib/iwd/wifi_network.psk`. After that, only the PreSharedKey can be passed to the role in order to use the network.
