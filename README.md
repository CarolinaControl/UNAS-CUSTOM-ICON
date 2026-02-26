# 🖥️ macOS Custom Icon Fix for UniFi UNAS

A guide to transforming the generic "Blue Screen" PC icon in macOS Finder into a professional **RackMac** (Server) icon by modifying Samba and Avahi configurations.

---

## 🛠️ Prerequisites

* **SSH Access:** Enabled via the UniFi OS settings.
* **Terminal:** Access to a CLI (macOS Terminal, iTerm2, etc.).
* **Root/Sudo Privileges:** Required to modify system configuration files.

---

## 🚀 Implementation

### 1. Samba Configuration (`smb.conf`)
The `fruit` module is an Apple-specific extension for Samba that allows macOS to recognize specific device metadata and performance optimizations.

1.  **SSH into your UNAS:**
    ```bash
    ssh root@<your-unas-ip>
    ```
2.  **Open the Samba configuration file:**
    ```bash
    nano /etc/samba/smb.conf
    ```
3.  **Update the `[global]` section:**
    Ensure the following parameters are present to signal the hardware type to macOS:
    ```ini
    [global]
    vfs objects = fruit streams_xattr
    fruit:metadata = stream
    fruit:model = RackMac
    fruit:posix_rename = yes
    fruit:veto_appledouble = no
    fruit:nfs_aces = no
    ```
4.  **Restart the Samba service:**
    ```bash
    systemctl restart smbd
    ```

---

### 2. Avahi Service Broadcast (Bonjour)
macOS often prioritizes **mDNS (Bonjour)** for Finder icons. Adding a `device-info` record ensures the icon appears even before the drive is mounted.

1.  **Edit the SMB service file:**
    ```bash
    nano /etc/avahi/services/samba.service
    ```
2.  **Insert the `<txt-record>` for the model:**
    ```xml
    <?xml version="1.0" standalone='no'?>
    <!DOCTYPE service-group SYSTEM "avahi-service.dtd">
    <service-group>
      <name replace-wildcards="yes">%h</name>
      <service>
        <type>_smb._tcp</type>
        <port>445</port>
      </service>
      <service>
        <type>_
