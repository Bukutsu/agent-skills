# Build and release verification

Use this branch for build, CI, packaging, or release changes. Keep verification within the requested scope; this checklist does not authorize publishing or expanding the release process.

## Map the affected paths

Read version-matched build instructions and dependency declarations. Check installed headers, libraries, and runtime assets before requesting installation; give one consolidated list of known missing prerequisites. Use the documented build entry point, hooks, and feature flags.

List the affected local scripts, workflow jobs, platform variants, and generated artifacts. Follow each artifact from generation to tests, bundling, and delivery. Include separate workflows triggered by the same release when the request covers the whole release. A shared script or one passing platform does not establish that every caller forwards the same flags or runs prerequisites in the same order.

**Complete when:** each affected path has a producer/consumer order and a planned check, or an explicit environment limitation.

## Verify from clean inputs

Use a disposable checkout matching the candidate commit and intended diff. Install from its lockfiles and run the relevant workflow commands in their actual order, starting without locally generated assets. Keep writable build outputs, generated metadata, and application state separate from the active checkout. Reuse compatible isolated outputs for incremental checks, but label those as warm-cache checks rather than fresh-build evidence. Leave the user's existing artifacts in place.

Check that unit-test mocks stop at the intended boundary. If tests deliberately mock generated code, also verify the real generation and integration path where affected. A passing mocked suite does not establish that the shipped artifact contains its runtime dependencies.

For cross-platform changes, inspect every affected invocation and execute checks on available targets. Record unavailable platforms rather than extrapolating from the host build. YAML parsing establishes syntax only, not workflow expressions, shell behavior, action inputs, or artifact correctness.

**Complete when:** clean-input checks pass for the exercised paths, with missing platforms and integration checks listed. If isolation or required tools are unavailable, report the blocker instead of claiming clean-build verification.

## Track delivery evidence

When publication is authorized, bind evidence to the exact commit or tag, workflow run, and expected artifact or destination. Track separate release and deploy workflows separately. A successful desktop release does not establish a successful web deployment, and a run on an older tag does not validate fixes on the branch tip.

Read each required run's terminal result and verify its expected artifacts or destination before claiming delivery. If a run is still active or monitoring stops, report its ID, revision, and pending checks. Describe unexecuted workflow changes as locally checked or awaiting CI, not guaranteed to pass next time.

**Complete when:** every requested delivery target has revision-matched terminal evidence, or is explicitly failed, pending, or unverified.
