# Handlebars Templates

#patterns #handlebars #xml #templates

Handlebars rendering is available in function nodes via libs config. Used for XML generation in NetCHB.

## Availability

Handlebars npm module IS available in the function node runtime via libs config:

```json
"libs": [{"var": "Handlebars", "module": "handlebars"}]
```

Source: Confirmed working 2026-04-03, expertise.yaml handlebars_templates

## Pattern

1. `get-native-object` fetches template record by ID (from config-driven lookup)
2. Function node loads Handlebars, compiles template, renders with ECD data
3. Falls back to pure JS if template not available

```js
var Handlebars = libs.Handlebars;
var template = Handlebars.compile(msg.templateRecord.content);
var xml = template(msg.ecdData);
```

## Custom Helpers Registered

11 helpers in the NetCHB entry template:
`findParty`, `generateParties`, `sum`, `fixed`, `truncate`, `upper`, `eq`, `in`, `or`, `getParty`, `findRegistrationNumber`

## Template Storage

Templates stored as `xml-template` records in native object store. Template content is 441 lines for the entry type, 13,192 chars rendered output.

First successful render: ACCEPTED by NetCHB as ST9-2074822-7 (2026-04-03).

## Waybill Handling

xml-builder handles both house-bill and master-bill:
```js
if (ecd.waybillType === 'house') {
  // use <house-bill>
} else {
  // use <master-bill>
}
```

Source: expertise.yaml waybill_handling (still needs prod verification with Brad)

## SOAP Auth in XML

Per-call credentials in SOAP body. Password needs HTML entity encoding (`>` becomes `&gt;`). ISF earlier failure was bash truncating password at `>` character.

Source: expertise.yaml soap_auth, isf_sandbox_auth

## Related

- [[config-driven-routing]] -- how template ID is looked up
- [[http-api]] -- SOAP wrapping after template render
- [[clients/alba-netchb]] -- template records and filing history
