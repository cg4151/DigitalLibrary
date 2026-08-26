# Configuration Setup Guide

This guide explains how to configure the NSBM Library applications using environment variables instead of hardcoded values.

## Overview

The applications have been refactored to support environment-based configuration, making them portable across different deployment environments without code changes.

## Configuration Variables

### 1. Service Endpoints

| Variable | Default | Description |
|----------|---------|-------------|
| `NHRM_SERVICE_BASE_URL` | `https://10.10.10.24/nhrmbase` | Base URL for NHRMBase WCF service |
| `NHRM_PAYMENT_SERVICE_URL` | `https://10.10.10.24/nhrmbase/NHRMPayment.svc` | NHRMPayment service endpoint |
| `NHRM_UTILS_SERVICE_URL` | `https://10.10.10.24/nhrmbase/NHRMUtils.svc` | NHRMUtils service endpoint |

### 2. RFID Reader Configuration

| Variable | Default | Description |
|----------|---------|-------------|
| `RFID_READER_IP` | `127.0.0.1` | IP address of RFID reader |
| `RFID_READER_PORT` | `3050` | Port for RFID reader connection |

### 3. Logging Configuration

| Variable | Default | Description |
|----------|---------|-------------|
| `LOG_DIR` | `C:\logs\NSBM` | Base directory for application logs |
| `LOG_LEVEL` | `Info` | Logging level (Debug, Info, Warn, Error) |

## Setup Instructions

### Windows Development Environment

#### Option 1: System Environment Variables

1. Open **Environment Variables** (Win+R → `sysdm.cpl`)
2. Click **Environment Variables** → **New** (User or System)
3. Add each variable:
   ```
   NHRM_SERVICE_BASE_URL=https://your-server:port/nhrmbase
   RFID_READER_IP=192.168.1.100
   LOG_DIR=C:\logs\NSBM
   ```
4. Click **OK** and restart your application

#### Option 2: .env File (Recommended for Development)

1. Create a `.env` file in the application directory:
   ```
   NHRM_SERVICE_BASE_URL=https://localhost:8443/nhrmbase
   RFID_READER_IP=192.168.1.50
   LOG_DIR=C:\logs\NSBM\dev
   ```

2. Load it before running the application (see Application Setup below)

### Application Setup

#### For NHRMLIS Desktop App

1. Create `app.config.template` alongside `NHRMLIS.exe`
2. Before running the application, load environment variables:
   ```batch
   @echo off
   REM Load .env file if it exists
   if exist ".env" (
       for /f "delims== tokens=1,2" %%A in (.env) do (
           set %%A=%%B
       )
   )
   start NHRMLIS.exe
   ```

#### For NHRMLISWT (with Reader)

1. Update `Reader.ini` (see below)
2. Set environment variables as described above
3. Run the application

### Reader Configuration

**Reader.ini** now supports environment variable substitution:

```ini
[Reader]
IP=${RFID_READER_IP:127.0.0.1}
Port=${RFID_READER_PORT:3050}
```

The format `${VARIABLE:default_value}` means:
- Use the environment variable `VARIABLE` if set
- Otherwise use `default_value`

## Docker / Container Deployment

For containerized deployment, pass environment variables at runtime:

```bash
docker run -e NHRM_SERVICE_BASE_URL=https://prod-server/nhrmbase \
           -e RFID_READER_IP=10.0.0.50 \
           -e LOG_DIR=/var/log/nsbm \
           nsbm-library-app:latest
```

## Configuration Priority

Configurations are loaded in this order (later overrides earlier):

1. **Default values** in config files
2. **`.env` file** (if present)
3. **System Environment Variables** (highest priority)

## Security Best Practices

### DO
✅ Use environment variables for sensitive data (IPs, credentials)  
✅ Add `.env` and `*.local.config` to `.gitignore`  
✅ Use different configurations for dev/test/prod  
✅ Rotate credentials regularly  
✅ Use HTTPS for all service endpoints  

### DON'T
❌ Commit hardcoded IPs or credentials to git  
❌ Share `.env` files in repositories  
❌ Use the same credentials across environments  
❌ Log sensitive configuration values  

## Example Setups

### Development Environment
```bash
NHRM_SERVICE_BASE_URL=https://localhost:8443/nhrmbase
RFID_READER_IP=192.168.1.100
LOG_DIR=C:\logs\NSBM\dev
LOG_LEVEL=Debug
```

### Production Environment
```bash
NHRM_SERVICE_BASE_URL=https://prod-nhrm-server.company.com/nhrmbase
RFID_READER_IP=10.0.0.50
LOG_DIR=/var/log/nsbm/production
LOG_LEVEL=Info
```

### Testing Environment
```bash
NHRM_SERVICE_BASE_URL=https://test-nhrm-server.company.com/nhrmbase
RFID_READER_IP=192.168.100.50
LOG_DIR=C:\logs\NSBM\test
LOG_LEVEL=Warn
```

## Troubleshooting

**Issue:** Application using wrong IP address
- **Solution:** Verify environment variable is set with `echo %NHRM_SERVICE_BASE_URL%` (Windows) or `echo $NHRM_SERVICE_BASE_URL` (Linux/Mac)
- Restart application after changing variables

**Issue:** Logs not appearing in expected directory
- **Solution:** Check `LOG_DIR` is set and directory exists; application must have write permissions

**Issue:** Reader connection fails
- **Solution:** Verify `RFID_READER_IP` is correct and reader is on the network

## Migration Checklist

- [ ] Create `.env` files for each environment
- [ ] Update application config files to use environment variables
- [ ] Test each environment configuration
- [ ] Document any custom variables specific to your setup
- [ ] Train team on new configuration process
- [ ] Remove hardcoded values from version control
- [ ] Update deployment scripts/documentation
