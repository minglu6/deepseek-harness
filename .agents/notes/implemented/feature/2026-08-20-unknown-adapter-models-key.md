# Agent Note: Unknown adapter families get an API key on the Models page

Status: implemented

English | [中文](2026-08-20-unknown-adapter-models-key.zh.md)

## Problem

The Models editor keyed curated cards on `llm-deepseek` and `llm-pi-ai` by name. A third-party adapter that registered through `registerConfigurableProviders` appeared in Add provider — often first, in directory order — but selecting it showed only the `settings.yaml` hint, with Apply disabled and no key field. A plugin cannot inject that card; the page is the only configuration surface that matches other providers.

## Decision

An unknown settings namespace still renders the shared **API key** field. Apply stores the typed key through `credentials.set` under the profile's reference (or `<ROUTE>_API_KEY` when the profile has none) and materializes an empty profile when the schema has no `apiKeyEnv` to record. Remaining connection facts stay in `settings.yaml`. The Add-provider list is sorted by provider id so a separately registered plugin is not pinned at the top.

`llm-qoder` is a third curated family: PAT, optional VPC instance, and the same model catalog editor as DeepSeek. It does not show gateway or OpenAPI URLs. Other unknown adapters stay key-only.

## Alternatives considered

**Special-case each third-party namespace in the editor.** Rejected because every new plugin would need a harness UI change.

**Put the key on Settings → Plugins.** Rejected: that surface is for plugin tunables, not provider credentials, and does not match the Models page.

**Leave unknown families hint-only.** Rejected because a git-installed plugin then cannot be configured from the product UI.

**Sort configured rows as well.** Deferred: installed rows keep directory order; only the Add list was jumping a new plugin to the top.

## Consequences

Any adapter that declares a settings path can be given a key from Models. Extra fields (VPC, custom endpoints) remain yaml. A schema without `apiKeyEnv` still materializes a user profile so the row becomes configured and deletable.

## Testing

`packages/client/ui-settings-models/tests/components.client.spec.tsx` covers Add-list order, the unknown-family key field, and storing `PLAIN_API_KEY` while materializing the profile.
