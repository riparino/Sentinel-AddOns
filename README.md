# Microsoft Sentinel AI Incident Enrichment Logic App

This Logic App automatically enriches Microsoft Sentinel incidents using AI-powered KQL query generation and analysis.

## Overview

The workflow:
1. Triggers on new Sentinel incidents
2. Uses OpenAI GPT-4o-mini to generate relevant KQL queries  
3. Executes queries against Log Analytics
4. Analyzes results with AI
5. Adds enriched analysis as an incident comment

## Prerequisites

- Azure subscription with Microsoft Sentinel enabled
- Log Analytics workspace
- OpenAI API key
- System-assigned managed identity for the Logic App
- Azure Key Vault (recommended for secure secret storage)

## Setup Instructions

### 1. Prerequisites Setup

1. Create or identify your Log Analytics workspace
2. Ensure Microsoft Sentinel is enabled on the workspace
3. Note down:
   - Subscription ID
   - Resource Group name
   - Workspace name
   - Workspace ID

### 2. Create Logic App

1. Create a new Logic App in Azure Portal
2. Enable system-assigned managed identity:
   - Go to Logic App > Settings > Identity
   - Enable "System assigned" identity
   - Note the Object ID

### 3. Set up Azure Key Vault (Recommended)

For secure secret management, use Azure Key Vault:

1. Create an Azure Key Vault:
   ```bash
   az keyvault create --name [YOUR-KEYVAULT-NAME] --resource-group [YOUR-RESOURCE-GROUP] --location [YOUR-LOCATION]
   ```

2. Add your OpenAI API key to Key Vault:
   ```bash
   az keyvault secret set --vault-name [YOUR-KEYVAULT-NAME] --name "openai-api-key" --value "[YOUR-OPENAI-API-KEY]"
   ```

3. Grant the Logic App managed identity access to Key Vault:
   ```bash
   az keyvault set-policy --name [YOUR-KEYVAULT-NAME] --object-id [LOGIC-APP-OBJECT-ID] --secret-permissions get
   ```

4. Modify the Logic App to reference Key Vault:
   
   Instead of using the `openAiApiKey` parameter, use this in the Authorization header:
   ```json
   "Authorization": "Bearer @{body('Get_Secret')?['value']}"
   ```
   
   Add this action before the OpenAI API calls:
   ```json
   "Get_Secret": {
     "type": "Http",
     "inputs": {
       "uri": "https://[YOUR-KEYVAULT-NAME].vault.azure.net/secrets/openai-api-key?api-version=7.0",
       "method": "GET",
       "authentication": {
         "type": "ManagedServiceIdentity",
         "audience": "https://vault.azure.net"
       }
     }
   }
   ```

### 4. Grant Permissions

1. Assign Log Analytics Reader role to the Logic App's managed identity:
   ```bash
   az role assignment create --assignee [LOGIC-APP-OBJECT-ID] --role "Log Analytics Reader" --scope /subscriptions/[SUBSCRIPTION-ID]/resourceGroups/[RESOURCE-GROUP]/providers/Microsoft.OperationalInsights/workspaces/[WORKSPACE-NAME]
   ```

2. Grant Sentinel Contributor role for incident comments:
   ```bash
   az role assignment create --assignee [LOGIC-APP-OBJECT-ID] --role "Microsoft Sentinel Contributor" --scope /subscriptions/[SUBSCRIPTION-ID]/resourceGroups/[RESOURCE-GROUP]
   ```

### 5. Create API Connection

1. In the Logic App designer, create an "Azure Sentinel" connection
2. Select "Connect with managed identity"

### 6. Deploy the Logic App

1. Replace placeholders in the JSON file:
   - `[YOUR-SUBSCRIPTION-ID]`
   - `[YOUR-RESOURCE-GROUP]`
   - `[YOUR-WORKSPACE-NAME]`
   - `[YOUR-WORKSPACE-ID]`
   - `[YOUR-LOCATION]`

2. If not using Key Vault, also replace `[YOUR-OPENAI-API-KEY]`

3. Import the JSON into your Logic App

## Configuration Parameters

| Parameter | Description | Example |
|-----------|-------------|---------|
| subscriptionId | Azure subscription ID | `12345678-1234-1234-1234-123456789012` |
| resourceGroupName | Resource group containing Sentinel | `rg-sentinel` |
| workspaceName | Log Analytics workspace name | `log-sentinel-workspace` |
| workspaceId | Workspace GUID | `12345678-1234-1234-1234-123456789012` |
| openAiApiKey | OpenAI API key (if not using Key Vault) | Use Key Vault instead |

## Security Best Practices

1. **Never commit secrets**: Use Key Vault to store the OpenAI API key
2. **Least privilege access**: Only grant necessary permissions to the managed identity
3. **Network security**: Consider using private endpoints for Key Vault
4. **Monitor access**: Enable diagnostic logging for the Logic App and Key Vault
5. **Rotate keys**: Regularly rotate the OpenAI API key

## Troubleshooting

### Common Issues

1. **401 Unauthorized**: Verify managed identity has correct permissions
2. **403 Forbidden**: Check Key Vault access policies
3. **Query timeout**: Increase timeout settings or simplify KQL queries
4. **OpenAI rate limits**: Implement retry logic or upgrade OpenAI tier

### Debugging

1. Enable run history details:
   - Go to Logic App > Settings > Workflow settings
   - Enable "Run history" details

2. Check trigger history for unsuccessful runs
3. Review action outputs in run details
4. Monitor Logic App diagnostic logs

## Support

For issues or questions:
1. Check Azure Logic Apps documentation
2. Review Microsoft Sentinel documentation  
3. Consult OpenAI API documentation

## License

[MIT License](LICENSE)

## Contributors

Riparino

## Changelog

### Version 1.0.0
- Initial release
- Basic incident enrichment workflow
- OpenAI integration for query generation and analysis

## Acknowledgments

- Microsoft Sentinel team
- OpenAI API
