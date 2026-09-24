# OpenAPI Security Specification JSON Schema

This directory contains the YAML sources for generating the JSON Schemas for validating OpenAPI Security Specification (OSS) Standardized API Features (SAFs), which are published on [https://spec.openapis.org](https://spec.openapis.org).

***NOTE:*** The exact structure and versioning policy of SAF schemas is still being discussed.  The current schemas are ported over from the OpenAPI Specification but the approach is subject to change.

Due to limitations of GitHub pages, the schemas on the spec site are served with `Content-Type: application/octet-stream`, but should be interpreted as `application/schema+json`.

The sources in this directory, which have `WORK-IN-PROGRESS` in their `$id`s, are _not intended for direct use_.

## Schema `$id` dates

The published schemas on the spec site have an _iteration date_ in their `id`s.
This allows the schemas for a release line to be updated independent of the spec patch release cycle.

The iteration version of the JSON Schema can be found in the `$id` field.
For example, the value of `$id: https://spec.openapis.org/oas/3.1/schema/2021-03-02` means this iteration was created on March 2nd, 2021.

We are [working on](https://github.com/OAI/OpenAPI-Specification/issues/4152) how to best provide programmatic access for determining the latest date for each schema.

