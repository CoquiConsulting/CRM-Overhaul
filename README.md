# Healthcare Salesforce Org Consolidation & EHR Integration

Redesigned and rebuilt a legacy Salesforce environment for a multi-location healthcare provider. Consolidated three disconnected production orgs into a single unified platform, automated patient intake via Flow/Apex, and shipped a bi-directional REST integration with the provider's EHR system.

## Results

| Metric | Outcome |
|---|---|
| Manual data entry | Reduced by 70% |
| Org consolidation | 3 orgs → 1 unified org |
| EHR integration | Delivered in 6 weeks |

## Architecture Overview

The provider ran three independent Salesforce orgs (Org A, B, C), each with divergent custom object schemas, Process Builder/Workflow Rule automations, and field-level configurations. There was no shared `Patient` record across locations, resulting in duplicate data entry and no single source of truth.

This project re-platformed all three orgs onto one Salesforce instance with a normalized data model, declarative + programmatic automation for intake, and a custom integration layer to the EHR.

## Migration

- **Data audit**: Queried schema metadata (`describeSObjects`, field usage reports) across all three orgs to identify object/field overlaps, picklist mismatches, and orphaned records.
- **ETL pipeline**: Used the Bulk API via Data Loader (and custom batch jobs for complex transforms) to extract, transform, and load records into the target org.
- **De-duplication**: Implemented matching rules and Apex batch jobs (`Database.Batchable`) to merge duplicate `Account`/`Contact`/custom `Patient__c` records across the three source orgs based on MRN and demographic matching.
- **Schema consolidation**: Designed a unified object model — `Patient__c`, `Location__c`, `Care_Team__c` — with lookup/master-detail relationships replacing location-siloed custom objects.
- **Sharing model**: Implemented Role Hierarchy + Sharing Rules + Permission Sets to enforce location-based record access (RLS) post-consolidation.

## Patient Intake Automation

- **Record-Triggered Flows** replace manual intake forms — on `Patient__c` creation/update, Flow handles field validation, conditional routing, and related record creation (e.g. `Care_Episode__c`).
- **Apex Invocable Methods** are called from Flow for logic too complex for declarative tooling (e.g. cross-object validation, calling the EHR integration service).
- **Validation Rules** enforce required-field and data-format integrity at the database layer (e.g. NPI format, DOB constraints).
- **Asynchronous Apex** (`Queueable`/`Future`) offloads downstream processing (notifications, EHR sync triggers) to avoid governor limit issues on synchronous Flow paths.

## EHR Integration

- **Apex REST callouts** (`HttpRequest`/`HttpResponse`) authenticate against the EHR's API using a **Named Credential** + OAuth 2.0, avoiding hardcoded credentials/endpoints.
- **Outbound sync**: Platform Events (`EventBus.publish`) decouple Salesforce DML from the integration call, with a subscribing Apex trigger handling the actual EHR callout asynchronously.
- **Inbound sync**: Exposed Apex REST endpoints (`@RestResource`) for the EHR to push updates back into Salesforce, with payloads mapped via a custom Apex deserialization layer (`JSON.deserialize` into wrapper classes).
- **Error handling & retries**: Failed callouts are logged to a custom `Integration_Log__c` object and retried via scheduled Apex (`Schedulable` + `Queueable` chaining).
- Delivered end-to-end (auth, mapping, error handling, testing) in **6 weeks**.

## Tech Stack

- Salesforce Platform: Flow (record-triggered, screen flows), Apex (triggers, batch, queueable, REST resources), Custom Objects/Fields, Validation Rules, Platform Events
- Integration: REST/JSON, Named Credentials, OAuth 2.0, Apex callouts
- Data tooling: Bulk API, Data Loader, custom `Database.Batchable` jobs for migration and dedup
- Security: Role Hierarchy, Sharing Rules, Permission Sets, Field-Level Security

## Impact

- Eliminated duplicate manual data entry across three previously siloed systems
- 70% reduction in manual data entry per patient record via automated Flow/Apex intake
- Single normalized data model (`Patient__c`, `Location__c`, `Care_Team__c`) as system of record
- Real-time bi-directional EHR sync via Apex REST + Platform Events, replacing manual chart updates
- Integration delivered in 6 weeks from kickoff to production cutover
