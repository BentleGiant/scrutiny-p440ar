<p align="center">
  <a href="https://github.com/AnalogJ/scrutiny">
  <img width="300" alt="scrutiny_view" src="webapp/frontend/src/assets/images/logo/scrutiny-logo-dark.png">
  </a>
</p>


# Scrutiny - P440ar RAID Controller Fix

A fork of [Scrutiny](https://github.com/AnalogJ/scrutiny) with fixes for HP P440ar and other RAID controllers that require specific device type syntax.

[![CI](https://github.com/AnalogJ/scrutiny/workflows/CI/badge.svg?branch=master)](https://github.com/AnalogJ/scrutiny/actions?query=workflow%3ACI)
[![GitHub license](https://img.shields.io/github/license/AnalogJ/scrutiny.svg?style=flat-square)](https://github.com/AnalogJ/scrutiny/blob/master/LICENSE)

WebUI for smartd S.M.A.R.T monitoring

## What's Different in This Fork

### Device Type Preservation Fix
Preserve user-configured device types in `collector.yaml` instead of overwriting them with auto-detected values during metrics collection.

- **Problem solved**: HP P440ar/cciss controllers and other RAID controllers were showing false "Failed" status
- **Root cause**: Collector used configured device types correctly for detection but hardcoded `--device sat` for metrics collection
- **Impact**: Fixes GitHub issues [#618](https://github.com/AnalogJ/scrutiny/issues/618), [#680](https://github.com/AnalogJ/scrutiny/issues/680), [#700](https://github.com/AnalogJ/scrutiny/issues/700), [#759](https://github.com/AnalogJ/scrutiny/issues/759)
- **Backward compatible**: Auto-detection still works when no device type is configured
- **Benefits**: All RAID controllers requiring specific device types (cciss, megaraid, 3ware, etc.)

#### Technical Details
Modified `collector/pkg/detect/detect.go` to check if device type was explicitly configured before overwriting:

```go
// Only override DeviceType if it wasn't explicitly configured
// This preserves user-configured device types (e.g., cciss, megaraid) from collector.yaml
if len(device.DeviceType) == 0 {
    device.DeviceType = availableDeviceInfo.Device.Type
}
```

#### Example Configuration
```yaml
# collector.yaml
version: 1
devices:
  - device: /dev/sda
    type: ['cciss,0']  # Now preserved throughout collection
  - device: /dev/sdb
    type: ['cciss,1']
  - device: /dev/sdc
    type: ['cciss,2']
```

#### Before vs After
**Before (Original):**
```
INFO  Executing command: smartctl --info --json --device cciss,0 /dev/sda
INFO  Executing command: smartctl --xall --json --device sat /dev/sda  ❌
ERROR smartctl returned an error code (4) while processing sda
ERROR smartctl detected a checksum error
Result: Drive shows "Failed" status
```

**After (This Fork):**
```
INFO  Executing command: smartctl --info --json --device cciss,0 /dev/sda
INFO  Executing command: smartctl --xall --json --device cciss,0 /dev/sda  ✅
INFO  Publishing smartctl results for 0x5000cca0d8cd0c37
Result: Drive shows "Passed" status
```

### Docker Images
Pre-built Docker images with the fix are available via GitHub Container Registry:

```bash
# Collector only
docker pull ghcr.io/bentlegiant/scrutiny-p440ar:master-collector

# Web UI only
docker pull ghcr.io/bentlegiant/scrutiny-p440ar:master-web

# All-in-one
docker pull ghcr.io/bentlegiant/scrutiny-p440ar:master-omnibus
```

See [DOCKER-USAGE.md](.claude/DOCKER-USAGE.md) for complete deployment instructions.

# Licenses

- MIT
- Logo: [Glasses by matias porta lezcano](https://thenounproject.com/term/glasses/775232)
