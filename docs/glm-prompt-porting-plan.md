# GLM Prompt Variant Porting Plan

## Objective
Transfer the recently added GLM-4.6 prompt-optimization workflow (and supporting OpenRouter metadata) from this repository into another fork so that open-source models such as `glm-4.6` receive the tuned "thinking" prompt behavior.

## High-Level Components to Audit/Port
1. **Prompt Variant Registration** – Adds a dedicated `ModelFamily.GLM` branch in the variant registry so GLM-class models load their own system prompt. 【F:src/core/prompts/system-prompt/variants/index.ts†L9-L82】
2. **GLM-Specific Prompt Assets** – Includes the variant config, template, and component overrides tuned for iterative exploration, tool gating, and MCP usage guidance. 【F:src/core/prompts/system-prompt/variants/glm/config.ts†L1-L72】【F:src/core/prompts/system-prompt/variants/glm/overrides.ts†L1-L196】【F:src/core/prompts/system-prompt/variants/glm/template.ts†L1-L20】
3. **Model Detection Utilities** – Recognizes GLM IDs (e.g., `glm-4.6`, `z-ai/glm-4.6`) so the prompt selector routes to the new variant. 【F:src/utils/model-utils.ts†L1-L76】
4. **Model Metadata & Provider Preferences** – Registers GLM SKUs, defaults, and provider-ordering metadata for OpenRouter/Cerebras/Zhipu integrations. 【F:src/shared/api.ts†L739-L813】【F:src/shared/api.ts†L3022-L3688】
5. **UI Surfacing** – Highlights GLM-4.6 as a featured choice in the settings picker to guide users toward the optimized prompt. 【F:webview-ui/src/components/settings/OpenRouterModelPicker.tsx†L33-L87】
6. **Test Coverage** – Adds snapshot coverage ensuring the GLM variant composes correctly across context permutations. 【F:src/core/prompts/system-prompt/__tests__/integration.test.ts†L205-L240】

## Detailed Porting Plan

### File-Level Change Map
| File | Purpose | Key Actions |
| --- | --- | --- |
| `src/core/prompts/system-prompt/variants/index.ts` | Registers all system prompt variants and routes models to their respective configs. | Add a `ModelFamily.GLM` entry to `VARIANT_CONFIGS`, ensure `loadSystemPromptVariant` handles the new key, and update any related type unions. |
| `src/core/prompts/system-prompt/variants/glm/config.ts` | Defines the GLM prompt configuration object consumed by the registry. | Copy the exported `GLM_VARIANT_CONFIG`, preserving placeholder ordering, tool gating toggles, and MCP flag overrides. |
| `src/core/prompts/system-prompt/variants/glm/overrides.ts` | Customizes system prompt sections (thinking, reasoning, safety) for GLM. | Migrate the full `GLM_PROMPT_OVERRIDES` object, including section-specific Markdown strings and metadata hints. |
| `src/core/prompts/system-prompt/variants/glm/template.ts` | Supplies the base template for GLM prompt assembly. | Transfer the template literal and confirm helper imports (e.g., `stripIndent`) resolve in the fork. |
| `src/utils/model-utils.ts` | Contains helpers for model family detection. | Port the `isGLMModelFamily` predicate (and related regex/ID lists) to keep routing logic consistent. |
| `src/shared/api.ts` | Central model metadata registry for providers and defaults. | Copy GLM-specific provider entries, pricing metadata, and default ID exports referenced by the picker/UI. |
| `webview-ui/src/components/settings/OpenRouterModelPicker.tsx` | Renders the OpenRouter model selection UI. | Ensure GLM entries appear in the featured/trending sections and respect filtering logic. |
| `src/core/prompts/system-prompt/__tests__/integration.test.ts` | Snapshot coverage for prompt variants. | Import the GLM variant test case, update snapshots, and verify expectations align with the fork’s tooling. |

### 1. Align Shared Enumerations & Utilities
- Ensure the target fork exposes a `ModelFamily.GLM` enum entry and exports it through any prompt-related typing modules.
- Mirror the `isGLMModelFamily` helper so variant matchers can detect GLM SKUs. If the fork already normalizes model IDs differently, adjust the helper accordingly.

### 2. Introduce the GLM Prompt Variant Package
- Replicate the `variants/glm/` folder (config, template, overrides). These files rely on the shared `VariantBuilder`, placeholder enums, and default tool identifiers—confirm parity of these dependencies in the fork.
- If the fork lacks any of the referenced `SystemPromptSection` placeholders or `ClineDefaultTool` entries, add or map equivalents before integrating the GLM config.
- Re-run or regenerate any prompt-schema validation the fork enforces (e.g., `validateVariant`).

### 3. Register Variant in the Prompt Registry
- Update the fork’s `variants/index.ts` (or equivalent registry) to export the GLM config and include it in the `VARIANT_CONFIGS` map keyed by `ModelFamily.GLM`.
- Verify downstream code (prompt loader, provider switchers) respects the new map entry. Resolve any TypeScript exhaustiveness checks caused by the new key.

### 4. Extend Model Metadata for GLM Families
- Copy the GLM-specific blocks from `src/shared/api.ts`, including:
  - OpenRouter provider-preference orderings for `z-ai/glm-4.6` and related SKUs.
  - Model definition objects for Cerebras and Zhipu (international/mainland) integrations with pricing and context metadata.
- Align type exports (e.g., `CerebrasModelId`, `internationalZAiModelId`) if the fork renamed or scoped these differently.
- Confirm that default model IDs (e.g., `cerebrasDefaultModelId = "zai-glm-4.6"`) map to actual entries in the fork’s configuration.

### 5. Surface GLM Models in Front-End Pickers
- Update any OpenRouter or provider model pickers to feature `z-ai/glm-4.6:exacto` (and variants) so users can easily select the tuned prompt. The upstream UI marks it as "Trending"—preserve or adapt badges per design guidelines.
- Ensure the picker’s filtering rules continue to include the GLM IDs after porting.

### 6. Sync Tests and Snapshots
- Bring over the GLM entry in prompt integration tests. If the fork uses Jest snapshots (`__snapshots__`), regenerate them after introducing the variant to capture GLM-specific prompt output.
- Execute the prompt test suite (e.g., `npm run test -- system-prompt`) to confirm snapshots and variant validation pass.

### 7. Validation & Rollout
- After porting, run lint/build pipelines that validate shared prompt assets.
- Verify the UI renders the new featured model entry and that switching to GLM models triggers the GLM system prompt via manual or automated smoke testing.
- Document any provider credential prerequisites (e.g., Zhipu keys) in the fork’s README or deployment notes.

## Additional Considerations
- **Version Control:** Keep variant files versioned independently so future GLM prompt updates can be diffed cleanly across forks.
- **Localization:** If the fork supports localized prompt strings, audit whether GLM-specific templates require translation hooks.
- **Telemetry/Analytics:** Ensure any metrics keyed by model family account for the new `glm` identifier to avoid aggregation gaps.

## Next Steps Checklist
- [ ] Confirm enum/util support for `ModelFamily.GLM` in the fork.
- [ ] Copy `variants/glm/` assets and resolve dependency imports.
- [ ] Register the variant in the prompt registry.
- [ ] Import GLM model metadata into shared API layers.
- [ ] Update front-end model pickers or provider UIs.
- [ ] Run prompt integration tests and update snapshots.
- [ ] Perform manual verification and document provider requirements.
