# Repository guidance

This repository maintains reusable graphics-development skills and Chinese usage documentation.

- Keep each skill under `skills/<name>/SKILL.md` with a precise YAML name and description. Put conditional detail in linked references.
- Read the affected skill and its source notes before editing. Preserve external runner ownership: do not vendor Metallic tools, NVIDIA SDKs, captures, or Unity generated artifacts.
- Distinguish an imported snapshot, an extracted workflow, a historical observation, and current runtime validation. Never promote historical numbers or a proposed optimization to a verified result.
- Read the target project's current AGENTS.md when executing a workflow; the archived workflow does not override it.
- Update the root index and usage/source notes when adding or substantially changing a skill. Do not silently sync changes into globally installed skills.
- For documentation-only work validate frontmatter, links, and diff. Do not launch Unity, GPU benchmarks, or a full renderer build merely to validate these documents.
