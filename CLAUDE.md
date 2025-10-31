# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

This is the **XrayR-release** repository - a deployment and distribution repository for XrayR, not the source code repository. The actual source code is maintained at [silentdspeedup/NewXrayR](https://github.com/silentdspeedup/NewXrayR).

**Note: Docker functionality has been disabled in this repository.**

This repository contains:
- Installation scripts for Linux systems (CentOS, Ubuntu, Debian)
- Configuration files and templates
- ~~Docker deployment configuration~~ (disabled)
- Pre-built binaries and GeoIP/GeoSite databases

## Project Overview

XrayR is a backend framework for Xray that supports multiple panel types (SSpanel, V2board, NewV2board, PMpanel, Proxypanel, V2RaySocks) and protocols (V2ray, Vmess, Vless, Shadowsocks, Trojan).

## Key Files Structure

```
├── install.sh                      # Main installation script (downloads and installs XrayR)
├── XrayR.sh                        # Service management script (start/stop/update/etc.)
├── XrayR.service                   # Systemd service unit file
├── docker-compose.yml              # Docker Compose configuration
├── config/
│   ├── config.yml                  # Main configuration file (nodes, panels, connections)
│   ├── dns.json                    # Custom DNS configuration
│   ├── route.json                  # Routing rules configuration
│   ├── custom_inbound.json         # Custom inbound proxy settings
│   ├── custom_outbound.json        # Custom outbound proxy settings
│   ├── rulelist                    # Domain/IP rule list
│   ├── geoip.dat                   # GeoIP database for routing
│   └── geosite.dat                 # GeoSite database for routing
```

## Installation Methods

### Standard Installation (Linux) - RECOMMENDED
```bash
bash <(curl -Ls https://raw.githubusercontent.com/XrayR-project/XrayR-release/master/install.sh)
```

The install.sh script:
- Detects OS (CentOS/Ubuntu/Debian) and architecture (x64/arm64/s390x)
- Requires root privileges
- Installs dependencies and downloads NewXrayR binary from silentdspeedup/NewXrayR
- Sets up systemd service
- Can install ACME for certificate management

### ~~Docker Installation~~ (DISABLED)
Docker functionality has been disabled in this repository. Use standard installation instead.

### ~~Docker Compose~~ (DISABLED)
Docker Compose functionality has been disabled in this repository. Use standard installation instead.

## Service Management (Using XrayR.sh)

The XrayR.sh script provides a menu-driven interface for:
- Installing/updating XrayR
- Starting/stopping/restarting the service
- Viewing logs
- Checking status
- Uninstalling

Standard systemd commands also work:
```bash
systemctl start XrayR
systemctl stop XrayR
systemctl restart XrayR
systemctl status XrayR
systemctl enable XrayR  # Auto-start on boot
```

## Configuration Structure

### Main Configuration (config/config.yml)

The configuration uses a multi-node architecture where each node connects to a panel:

**Log Settings**: Set log level (none/error/warning/info/debug) and paths

**Connection Settings**: Handshake timeout, idle timeout, buffer size

**Nodes Array**: Each node requires:
- `PanelType`: Panel backend type (SSpanel, V2board, NewV2board, PMpanel, Proxypanel, V2RaySocks)
- `ApiConfig`: API connection details
  - ApiHost: Panel API endpoint
  - ApiKey: Authentication key
  - NodeID: Node identifier in the panel
  - NodeType: Protocol type (V2ray, Vmess, Vless, Shadowsocks, Trojan, Shadowsocks-Plugin)
  - Timeout: API request timeout
  - EnableVless/EnableXTLS: Protocol feature flags
  - SpeedLimit/DeviceLimit: Local limits (0 = disabled)
  - RuleListPath: Path to local rule file

- `ControllerConfig`: Network and proxy settings
  - ListenIP/SendIP: Network interface binding
  - UpdatePeriodic: Node info sync interval (seconds)
  - EnableDNS/DNSType: DNS configuration
  - EnableProxyProtocol: For WebSocket/TCP
  - AutoSpeedLimitConfig: Automatic speed limiting based on usage
  - GlobalDeviceLimitConfig: Redis-based device limiting across multiple nodes
  - EnableFallback/FallBackConfigs: Traffic fallback (Trojan/Vless)
  - CertConfig: TLS certificate management
    - CertMode: none/file/http/tls/dns
    - CertDomain: Domain for certificate
    - Provider: DNS provider for ACME (if using dns mode)
    - DNSEnv: Provider-specific environment variables

**Multiple Nodes**: Can run multiple nodes simultaneously by adding additional entries in the Nodes array

### Advanced Configuration Files

- **dns.json**: Custom DNS servers and routing rules (follows Xray DNS format)
- **route.json**: Traffic routing rules (follows Xray routing format)
- **custom_inbound.json**: Additional inbound proxy configurations
- **custom_outbound.json**: Additional outbound proxy configurations
- **rulelist**: Domain/IP filtering rules (one per line)

Configuration paths are referenced in config.yml via:
- DnsConfigPath
- RouteConfigPath
- InboundConfigPath
- OutboundConfigPath
- RuleListPath (per-node)

## Architecture Notes

### Binary Installation
- Default installation path: `/usr/local/XrayR/`
- Binary location: `/usr/local/XrayR/XrayR`
- Config location: `/etc/XrayR/config.yml`
- Service managed via systemd at `/etc/systemd/system/XrayR.service`
- Binary downloaded from: `https://github.com/silentdspeedup/NewXrayR/releases`

### ~~Docker Deployment~~ (DISABLED)
Docker deployment has been disabled in this repository. Use binary installation instead.

### Multi-Panel Support
XrayR can simultaneously connect to multiple panel backends and manage multiple nodes with different protocols. Each node in the configuration operates independently with its own:
- Panel connection
- Protocol type (V2ray, Shadowsocks, Trojan, etc.)
- Certificate management
- Speed/device limits
- Update intervals

## Documentation

Official documentation: https://xrayr-project.github.io/XrayR-doc/

For Xray configuration details:
- DNS: https://xtls.github.io/config/dns.html
- Routing: https://xtls.github.io/config/routing.html
- Inbound: https://xtls.github.io/config/inbound.html
- Outbound: https://xtls.github.io/config/outbound.html
- Fallback: https://xtls.github.io/config/features/fallback.html

## Important Notes

1. **This is NOT the source code repository** - for source code modifications, see [silentdspeedup/NewXrayR](https://github.com/silentdspeedup/NewXrayR). This repository is adapted for NewXrayR fork.

2. **Configuration sensitivity**: config.yml contains sensitive data (API keys, domain names, DNS credentials). Never commit production configurations.

3. **Architecture support**: The install.sh script supports x86_64, arm64-v8a, and s390x architectures. 32-bit systems are not supported.

4. **OS requirements**:
   - CentOS 7+
   - Ubuntu 16+
   - Debian 8+

5. **Root privileges required**: Both installation and service management require root access.

6. **Certificate management**: When using CertMode=dns, ensure DNS provider credentials are correctly set in DNSEnv. See https://go-acme.github.io/lego/dns/ for supported providers.

7. **Docker functionality disabled**: All Docker-related functionality (docker-compose.yml, Docker installation instructions) has been disabled in this repository. Use standard binary installation via install.sh instead. The XrayR.sh management script displays a warning that it is "不适用于docker" (not applicable for docker).
