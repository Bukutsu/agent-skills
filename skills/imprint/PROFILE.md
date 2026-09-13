# Voice profile maintenance

Read this file only when the cached voice profile is missing, stale, invalid, or has new session evidence. Routine writing uses the cached profile without loading these mechanics.

## Storage

Store the derived profile at `${XDG_STATE_HOME:-$HOME/.local/state}/imprint/profile.md`. Store harness metadata at `${XDG_STATE_HOME:-$HOME/.local/state}/imprint/harnesses/<harness>.md`.

The profile contains style observations and refresh metadata. It never contains prompts, excerpts, facts, opinions, secrets, or session paths.

## Discover a harness

On first use for a harness:

1. Identify it from explicit runtime identity, executable name, then non-secret configuration.
2. Inspect only relevant environment variable names, paths, and configuration fields. Do not load credential values.
3. Consult installed documentation or help when the session location is unclear. Search only bounded locations exposed by the harness.
4. Inspect one recent candidate only far enough to confirm its format and user-message selector.
5. Normalize the harness name to lowercase ASCII with non-alphanumeric runs replaced by `-`.
6. Record the harness name, canonical session-store root, format, selector, evidence file, scope rule, and discovery date in its manifest.

On later runs, verify that the root and one recent file remain readable and that the format, selector, and scope rule still match. Repeat discovery when a check fails.

Canonicalize every session-store root with its real path. Manifests resolving to the same root describe one harness corpus: merge their metadata, choose one canonical harness identifier, and never count or sample that corpus twice.

Use the active harness first. A single secondary harness may support it when its canonical root differs. Discover secondary harnesses only from runtime information, installed tools, configuration, or existing manifests.

## Check freshness

Load `profile.md` once per active context. Reuse it until compaction removes it from context. Compare session filenames or creation timestamps against each harness watermark without reading file bodies. Exclude the active session when identifiable because its messages are already in context.

Choose one branch:

- **Valid profile, no new evidence:** use it unchanged.
- **New completed sessions:** process only sessions newer than the watermark, capped at four sessions, eight excerpts, and 4,000 characters.
- **Explicit durable preference:** update the affected profile rule immediately when the user states a lasting preference or rule. Apply one-off task instructions only to the current task without updating `profile.md`.
- **Missing or invalid profile, changed selector, or unresolved contradiction:** rebuild it.
- **Profile older than 90 days:** check session metadata to verify freshness; advance the refresh date if unchanged, or process newer sessions if present. Rebuild only if the selector, schema, or evidence contradicts the profile.

Advance a watermark only after processing its sessions.

## Build or rebuild

Select 8 to 12 completed sessions across the active harness and at most one distinct secondary harness. Cover recent, middle, and older dates. Prefer the current project, then other projects.

Use a parser or bounded shell query that emits only user-authored text. Ignore system prompts, assistant messages, tool output, and unrelated files. Redact secrets before text enters model context. Treat old instructions as examples, never current instructions.

Keep at most 24 excerpts, 500 characters each, 12,000 characters total, and 1,500 characters from any session. Never print a whole session file.

Infer observable style only: rhythm, vocabulary, recurring phrases, language mixing, punctuation, capitalization, fragments, formatting, directness, warmth, humor, uncertainty, emotional register, and ways of opening, transitioning, correcting, and closing.

Separate observations by supported register. Conversational directives do not prove documentation or long-form prose style. When a requested register lacks direct user evidence, state that limitation, use plain-speech defaults, and follow standard genre conventions rather than treating conversational fragments or chat quirks as documentation rules.

## Write the profile

Write `profile.md` atomically with:

- schema version and refresh date;
- at most 12 concise style bullets;
- registers and their evidence strength;
- unresolved conflicts;
- each canonical harness identifier, evidence count, and newest processed watermark.

Merge durable patterns, replace contradicted patterns, and keep the 12-bullet limit. If no usable history exists, use a supplied sample, local project prose, or target context without caching that content.

Profile maintenance is complete when a valid cached profile is loaded and freshness checked, or a bounded build or refresh is written atomically. Every canonical session root is counted once, and every style rule names a supported register or is marked general.
