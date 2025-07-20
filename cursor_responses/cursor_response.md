# Configuration Update: GRAPHANA_DASHBOARD Variable Support

## Changes Made to `chatbot_DLI.py`

I have successfully updated the `chatbot_DLI.py` file to read and use the `GRAPHANA_DASHBOARD` variable from the `config.yaml` file. This allows for dynamic configuration of the Grafana dashboard URL instead of using the hardcoded value "3yqhu0mm9".

### Specific Changes:

1. **Configuration Loading Enhancement** (Line ~55):
   ```python
   # Added reading of GRAPHANA_DASHBOARD from config
   GRAPHANA_DASHBOARD = config_file['GRAPHANA_DASHBOARD']
   ```

2. **Dashboard URL Function Update** (Line ~142):
   ```python
   def get_grafana_dashboard_url():
       """Get the Grafana dashboard URL for embedding (cloud version)"""
       return f"https://9002-{GRAPHANA_DASHBOARD}.brevlab.com/d/5g-metrics/5g-network-metrics-dashboard?orgId=1&refresh=5s&theme=dark"
   ```

3. **Health Check URL Update** (Line ~183):
   ```python
   # Updated the health check to use the configured dashboard variable
   response = requests.get(f"https://9002-{GRAPHANA_DASHBOARD}.brevlab.com", timeout=5)
   ```

### Configuration File Structure:

The `config.yaml` file already contains the `GRAPHANA_DASHBOARD` variable:
```yaml
GRAPHANA_DASHBOARD: ""
```

### How to Use:

To use this feature, simply update the `GRAPHANA_DASHBOARD` value in the `config.yaml` file with the appropriate identifier that should replace "3yqhu0mm9" in the URL. For example:

```yaml
GRAPHANA_DASHBOARD: "abc123def"
```

This would result in URLs like:
- `https://9002-abc123def.brevlab.com/d/5g-metrics/...` (for the dashboard)
- `https://9002-abc123def.brevlab.com` (for health checks)

### Benefits:

1. **Flexibility**: The Grafana dashboard URL can now be configured without modifying the code
2. **Environment-specific**: Different environments can use different dashboard identifiers
3. **Maintainability**: URL changes can be managed through configuration
4. **Consistency**: Both the health check and dashboard embedding use the same configurable URL base

### Files Modified:
- `agentic-llm/chatbot_DLI.py` - Updated to read and use GRAPHANA_DASHBOARD variable

All changes have been successfully applied and the application will now use the configurable dashboard URL from the config file. 