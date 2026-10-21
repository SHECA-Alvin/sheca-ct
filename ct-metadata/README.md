# Chrome CT Application Metadata

This directory contains the production operator JSON and metadata snapshots for five logs. The data was retrieved from each production log's public `log.v3.json` endpoint. No new keys were generated, and no MMD values or status timestamps were inferred.

## Files

- `operator.json`: identifies the operator as SHECA and references the five existing public `log.v3.json` URLs.
- `logs/ct-*.json`: local snapshots of each production log's public metadata, preserving the original fields.
- `schemas/`: Chrome's current public operator and log JSON schemas as downloaded for validation.
- `validation-report.json`: source URLs and validation results for each log.

## Validation Results

The operator JSON and all five log metadata files passed validation against the downloaded official Chrome JSON schemas, including format checks. Additional checks confirmed that each public key's SHA-256 hash matches its Log ID, shard intervals are contiguous, and endpoints match the configuration. All five logs declare an MMD of 60 seconds.

## Usage

The operator JSON references existing public `log.v3.json` endpoints directly, so the local snapshots do not need separate public URLs. Before applying, publish `operator.json` at a stable, publicly accessible HTTPS URL and provide that URL in the Chromium log operator application. These files have only been generated locally and have not yet been published or pushed.

The operator name is currently SHECA, matching the project. Confirm that this is the name to use in the application. Provide the contact email address and authorized representatives in the Chromium application; these are not required fields in the current operator schema.

The local log files are snapshots and should be refreshed when the public endpoints change. Successful retrieval and schema validation do not establish Chrome policy compliance, nor do they verify submission API behavior, ongoing availability, or acceptance of Google's Merge Delay Monitor Root.

## Official References

- https://googlechrome.github.io/CertificateTransparency/log_policy.html
- https://googlechrome.github.io/CertificateTransparency/operator_list_schema_v1.json
- https://googlechrome.github.io/CertificateTransparency/log_schema_v2.json
