# Configuration Migration Checklist

This checklist guides you through migrating from hardcoded configuration to environment-based configuration.

## Phase 1: Preparation

- [ ] Review current configuration files
  - [ ] `NSBM Library App/NHRMLIS.exe.config`
  - [ ] `NSBM Library App/config.ini`
  - [ ] `NSBM Library App/Reader.ini`
  - [ ] `NHRMLISWT/NHRMLISWT.exe.config`
  - [ ] `NHRMLISWT/Reader.ini`

- [ ] Identify all hardcoded values:
  - [ ] Server IPs and URLs
  - [ ] Port numbers
  - [ ] Database credentials
  - [ ] Log directories
  - [ ] Reader IPs

- [ ] Document current environment setups
  - [ ] Development servers
  - [ ] Testing servers
  - [ ] Production servers

## Phase 2: Configuration Refactoring

### NHRMLIS Desktop Application

- [ ] Create `app.config.template` from current `NHRMLIS.exe.config`
- [ ] Identify service endpoints needing environment variable support
- [ ] Add comments documenting each environment variable
- [ ] Create `config.ini.template` from current `config.ini`
- [ ] Create `Reader.ini.template` from current `Reader.ini`
- [ ] Update `.gitignore` to exclude:
  - [ ] `*.exe.config` (keep only `.template`)
  - [ ] `config.ini` (keep only `.template`)
  - [ ] `Reader.ini` (keep only `.template`)
  - [ ] `*.local.config`

### NHRMLISWT Anti-Theft Application

- [ ] Create `app.config.template` from current `NHRMLISWT.exe.config`
- [ ] Create `Reader.ini.template` from current `Reader.ini`
- [ ] Update references in code to read from environment variables
- [ ] Update `.gitignore` to exclude local config files

## Phase 3: Application Code Updates

### C# / VB.NET Code Changes

- [ ] Create helper class for configuration management:
  ```csharp
  public static class ConfigHelper
  {
      public static string GetServiceUrl(string envVar, string defaultValue)
      {
          return Environment.GetEnvironmentVariable(envVar) ?? defaultValue;
      }
  }
  ```

- [ ] Update NHRMLIS startup code:
  - [ ] Replace hardcoded `10.10.10.24` with `Environment.GetEnvironmentVariable("NHRM_SERVICE_BASE_URL")`
  - [ ] Add fallback to config file if env var not set

- [ ] Update NHRMLISWT startup code:
  - [ ] Replace hardcoded service URL with environment variable
  - [ ] Update Reader.ini parsing to support env var substitution

- [ ] Add logging of configuration sources (for debugging)

## Phase 4: Testing

### Development Environment

- [ ] Create `.env` file with development settings
- [ ] Test NHRMLIS with environment variables set
  - [ ] Verify correct service URL is used
  - [ ] Verify logs appear in expected directory
- [ ] Test NHRMLISWT with environment variables set
  - [ ] Verify reader connection works
  - [ ] Verify service endpoint is correct

### Test Environment

- [ ] Create separate `.env.test` configuration
- [ ] Deploy application to test server
- [ ] Verify configuration loads correctly
- [ ] Test with different environment variable values
- [ ] Verify no hardcoded values in production

### Production Preparation

- [ ] Create `.env.production` configuration (for reference only, store securely)
- [ ] Document production environment variables
- [ ] Set up environment variables on production servers:
  - [ ] Windows: via System Properties → Environment Variables
  - [ ] Linux: via `/etc/environment` or systemd service file
  - [ ] Docker: via `-e` flags or .env files

## Phase 5: Documentation

- [ ] Complete `CONFIG_SETUP.md` with examples
- [ ] Document each environment variable:
  - [ ] Purpose
  - [ ] Default value
  - [ ] Valid values/constraints
  - [ ] Example values for dev/test/prod

- [ ] Create deployment guide:
  - [ ] How to set up new environment
  - [ ] How to validate configuration
  - [ ] Troubleshooting guide

- [ ] Document fallback behavior:
  - [ ] What happens if env var not set
  - [ ] Order of configuration precedence

## Phase 6: Deployment

### Development Team

- [ ] Distribute `.env.example` to team
- [ ] Train team on new configuration process
- [ ] Update development setup documentation
- [ ] Verify all team members can run applications

### Production Deployment

- [ ] Create deployment script that sets environment variables
- [ ] Document environment variable setup for IT/DevOps team
- [ ] Verify production deployment works
- [ ] Remove old hardcoded config files from production
- [ ] Set up monitoring to alert on configuration issues

## Phase 7: Cleanup

- [ ] Remove old hardcoded config files from git history (careful with secrets)
  ```bash
  git filter-branch --tree-filter 'rm -f *.exe.config config.ini Reader.ini' HEAD
  ```
  **Note:** Only do this if no sensitive data was committed

- [ ] Archive old configuration for reference
- [ ] Update team documentation
- [ ] Mark old deployment procedures as deprecated

## Phase 8: Verification

- [ ] Run applications in all environments:
  - [ ] Development: Uses dev `.env`
  - [ ] Test: Uses test environment variables
  - [ ] Production: Uses production environment variables

- [ ] Verify functionality:
  - [ ] NHRMLIS connects to correct NHRMBase service
  - [ ] NHRMLISWT detects reader on correct IP
  - [ ] Logs write to correct directories
  - [ ] No hardcoded values in logs or error messages

- [ ] Performance testing:
  - [ ] Confirm no performance degradation
  - [ ] Check configuration loading time

## Known Issues & Workarounds

### Issue: .NET 4.5 doesn't support reading environment variables in app.config

**Solution:** Read environment variables programmatically in application startup

```csharp
public static void InitializeConfiguration()
{
    string baseUrl = Environment.GetEnvironmentVariable("NHRM_SERVICE_BASE_URL") 
        ?? "https://10.10.10.24/nhrmbase";
    
    // Update WCF client endpoints
    UpdateServiceEndpoints(baseUrl);
}
```

### Issue: Reader.ini is read before environment variables are set

**Solution:** Load `.env` file before parsing Reader.ini

```csharp
LoadDotEnvFile();
string readerIp = Environment.GetEnvironmentVariable("RFID_READER_IP") ?? "127.0.0.1";
```

## Rollback Plan

If issues arise with the new configuration system:

1. Revert to previous branch
2. Restore old config files from backup
3. Investigate configuration loading in application code
4. Fix and test in development before re-deploying

## Success Criteria

✅ All applications run with environment-based configuration  
✅ No hardcoded IPs or credentials in repository  
✅ Configuration can be changed without recompilation  
✅ Documentation is complete and team is trained  
✅ All environments (dev/test/prod) work correctly  
✅ Configuration is secure and audit-able  

## Support & Questions

For questions or issues during migration, refer to:
- `CONFIG_SETUP.md` for configuration details
- `.env.example` for available variables
- Application logs for configuration errors
