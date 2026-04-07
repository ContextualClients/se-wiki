# JH

#people #r-and-d #architecture #direction #cto

Platform leadership / architecture owner. Defines product direction for SE builds. Sets scope and approach for connectors.

**Title:** Chief Technology Officer
**Department:** R&D
**Reports to:** CEO
**Tenure:** 1 year 11 months

## Role

Internal Contextual. CTO leading R&D. Reports directly to CEO. Provides direction on architecture decisions, product scope, and SE build approach. Has ability to grant prod access. The Engineering org (AB as VP of Engineering) is a separate branch also reporting to CEO -- AB's team handles platform engineering delivery.

## Key Directions (2026-04-03)

From "Sync on Design Approach" session (transcript at docs/meetingnotes.md):

**Prod access:** "nothing keeping you from read-only access to prod, I can set that up"

**Filing types scope:** ALL types through NetCHB, including ISF and beyond type 01. Long-term: 8 entry types + ISF, AMS, In-Bond, FTZ.

**Template approach:** Config-driven XML template lookup by `(clientId, objectType)`. Not hardcoded per type. Multiple object types can map to different templates.

**ECD schema:** No changes for now.

**Entry types to support:**
- 01 Formal (current)
- 02 Quota
- 03 AD/CVD
- 06 FTZ
- 11 Informal
- 21 Warehouse Entry
- 23 TIB
- 31 Warehouse Withdrawal

Source: project_jeff_direction_20260403.md, expertise.yaml architecture_direction

## Platform Bug Confirmation

Confirmed `msg.objectId` reserved name bug in native object nodes (2026-04-03). Bug filed in Jira. Nasser also confirmed.

See [[platform/native-object-nodes]] for the full bug detail and workaround.

## Architecture Direction

From "Sync on Design Approach" meeting 2026-04-02 with AB:
- Standard object: Extended Customs Declaration for ALL importers
- Three-part connector model (outbound XML, internal API, inbound events)
- Schema rule: never create from scratch, always pull from reference tenant

## Related

- [[clients/alba-netchb]] -- implementation of Jeff's direction
- [[patterns/config-driven-routing]] -- template config approach
- [[platform/native-object-nodes]] -- bug Jeff confirmed
- [[people/ab]] -- VP of Engineering, peer at CEO-report level
- [[people/js]] -- Platform Engineering (under AL, under AB)
- [[people/aw]] -- Director of SE (Delivery org, reports to PK)
