> **Retired.** For live cake-autorate metrics, use the upstream [`mqtt-publisher.sh`](https://github.com/lynxthecat/cake-autorate/blob/master/mqtt-publisher.sh), which publishes them to MQTT and Home Assistant. This script still works as a minimal example of a CGI status endpoint on OpenWrt, but it isn't maintained.

# openwrt-cake-autorate-status

A minimal CGI endpoint for [cake-autorate](https://github.com/lynxthecat/cake-autorate) running on OpenWrt.
Exposes real-time CAKE shaper state as JSON so it can be consumed by dashboards (e.g. [Homepage](https://gethomepage.dev/)).

## JSON output

```json
{
  "timestamp": "2026-04-03T01:23:45",
  "dl_achieved_mbps": 441.2,
  "ul_achieved_mbps": 72.4,
  "dl_shaper_mbps": 487.0,
  "ul_shaper_mbps": 89.0,
  "dl_owd_delta_ms": 12.3,
  "ul_owd_delta_ms": 8.7,
  "dl_condition": "GOOD",
  "ul_condition": "GOOD",
  "tc_dl": "487Mbit",
  "tc_ul": "89Mbit"
}
```

Fields:
| Field | Description |
|---|---|
| `timestamp` | Timestamp of last SUMMARY line in cake-autorate log |
| `dl_achieved_mbps` | Measured download throughput (Mbit/s) |
| `ul_achieved_mbps` | Measured upload throughput (Mbit/s) |
| `dl_shaper_mbps` | cake-autorate's current DL shaper rate (Mbit/s) |
| `ul_shaper_mbps` | cake-autorate's current UL shaper rate (Mbit/s) |
| `dl_owd_delta_ms` | Download OWD delta (ms) — bufferbloat indicator |
| `ul_owd_delta_ms` | Upload OWD delta (ms) — bufferbloat indicator |
| `dl_condition` | DL link condition: `GOOD`, `BUFFERBLOAT`, `IDLE`, `RECOVERED` |
| `ul_condition` | UL link condition: same values |
| `tc_dl` | Kernel-confirmed DL shaper bandwidth (from `tc qdisc`) |
| `tc_ul` | Kernel-confirmed UL shaper bandwidth (from `tc qdisc`) |

## Requirements

- OpenWrt with `uhttpd` and CGI enabled (`cgi_prefix '/cgi-bin'`)
- `cake-autorate` running and logging to `/tmp/log/cake-autorate.primary.log`
- IFB interface name matching your CAKE setup (default: `ifb4eth1` for DL, `eth1` for UL)

## Install

```sh
# Copy the CGI script to OpenWrt
scp cgi-bin/cake-autorate-status root@192.168.8.1:/www/cgi-bin/cake-autorate-status
ssh root@192.168.8.1 'chmod +x /www/cgi-bin/cake-autorate-status'

# Persist across sysupgrade
ssh root@192.168.8.1 'echo /www/cgi-bin/cake-autorate-status >> /etc/sysupgrade.conf'
```

### Adjust interface names

Edit the top of `cgi-bin/cake-autorate-status` to match your setup:

```sh
DL_IF=ifb4eth1   # IFB interface for download (CAKE ingress)
UL_IF=eth1       # WAN interface for upload (CAKE egress)
```

Check your actual interfaces with:
```sh
tc qdisc show | grep cake
```

### Test

```sh
curl http://192.168.8.1/cgi-bin/cake-autorate-status
```

## Homepage widget

See [`examples/homepage-widget.yaml`](examples/homepage-widget.yaml) for a ready-to-use `customapi` widget config.

## Future plans

- LuCI app (status page in OpenWrt web UI)
- OpenWrt package / ipk
- Prometheus exporter
