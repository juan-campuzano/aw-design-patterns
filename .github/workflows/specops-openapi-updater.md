---
on:
  push:
    paths:
      - "src/routes/**"
    branches:
      - main

permissions:
  contents: read
  issues: read
  pull-requests: read

safe-outputs:
  create-pull-request:
    title-prefix: "[SpecOps] Update OpenAPI spec — "
    draft: false

tools:
  filesystem: {}
  github: {}
---

# OpenAPI Specification Updater

A push to `src/routes/` was detected. Update the OpenAPI specification to match.

## Instructions

1. **Read current spec**: Load `docs/openapi.yaml` as the baseline.

2. **Scan changed route files**: Examine the diff for new, modified, or deleted routes.
   Look for:
   - HTTP method and path
   - Request body schema (TypeScript types, Zod schemas, Pydantic models)
   - Response types and status codes
   - Authentication requirements (Bearer, API key, none)
   - Query parameters and path variables

3. **Update the spec**: Modify `docs/openapi.yaml` to reflect changes.
   - Follow existing naming conventions and style.
   - Use `$ref` for shared schemas where possible.
   - Include `summary` and `description` for all new endpoints.
   - Mark deprecated endpoints with `deprecated: true`.

4. **Validate**: Ensure the resulting YAML is valid OpenAPI 3.1.

5. **Open a PR** with the spec changes. Title should mention which routes changed.
