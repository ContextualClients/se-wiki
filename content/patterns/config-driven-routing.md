# Config-Driven Routing

#patterns #config #templates #routing #jeff

Routing logic lives in config records, not in flow code. Enables multi-client, multi-entry-type support without flow changes.

## The Pattern

1. Event fires (ECD/ISF post-insert)
2. Agent reads `netchb-template-config` record by `(clientId, objectType)`
3. Config record contains: XML template ID, target endpoint, SOAP operation
4. Agent fetches XML template, renders it, sends to correct endpoint

No hardcoded routing in flow code.

Source: JH 2026-04-03, expertise.yaml config_driven_templates

## Config Record Structure (netchb-template-config)

```json
{
  "clientId": "walmart",
  "objectType": "extended-customs-declaration",
  "templateId": "entry-template-01",
  "endpoint": "https://sandbox.netchb.com/main/services/entry/EntryUploadService",
  "operation": "uploadEntry"
}
```

11 records seeded for current filing types.

## Entry Types to Support Long-Term

Per Jeff 2026-04-03:
- 01 Formal (current, working)
- 02 Quota
- 03 AD/CVD
- 06 FTZ
- 11 Informal
- 21 Warehouse Entry
- 23 TIB
- 31 Warehouse Withdrawal
- Plus: ISF, AMS, In-Bond, FTZ

Each maps to a different template and potentially a different endpoint.

## Why Config-Driven

The connector itself stays the same -- it just routes to the right template and endpoint. Adding a new entry type = add a config record and a template record. No flow deploy needed.

## SOAP Endpoint Mapping

```
Entry: https://sandbox.netchb.com/main/services/entry/EntryUploadService
ISF:   https://sandbox.netchb.com/main/services/isf/IsfUploadService
```

SOAPAction is empty string `""` for ALL operations.

Source: expertise.yaml netchb_sandbox, soap_auth

## Related

- [[handlebars-templates]] -- how templates are rendered
- [[http-api]] -- SOAP abstraction in netchb-connector
- [[clients/alba-netchb]] -- current netchb-template-config records
