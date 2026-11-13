# Portuguese contribution security review

Reviewed on 2026-11-13 against fork PR #1 and the tracked tree based on upstream v1.131.0. This is a focused review of the contribution and packaged files, not a penetration test of Home Assistant or an exhaustive audit of upstream cards.

## Findings

- **No credential leaks detected.** Gitleaks v8.30.1, downloaded from its official GitHub release and checked against the published SHA-256, reported zero findings in the PR commit range (including merge changes), current tracked files, and local test packages, including archive contents.
- **No private configuration shipped.** The tracked filenames and ZIP contents contain no `.env`, Home Assistant configuration/secrets files, private keys, or credential files. The package text contains no developer home paths or private-network addresses. Documentation uses public links, loopback demo URLs and generic sensor names.
- **No added data-exfiltration path found.** Our executable changes add no network requests, telemetry, third-party runtime dependencies, token reads, or device-control calls. Existing network/service API call sites in the generated bundle match upstream.
- **Fixed a low-risk locale lookup edge case.** The new resolver could treat an inherited JavaScript property such as `constructor` as a language. It now checks own properties only. Regression tests cover both inherited object properties and inherited translation entries. This was an invalid-locale handling issue; the review found no associated credential exposure or code execution.
- **Commit metadata is public.** Author and committer identities are part of public Git history. They are separate from the shipped JavaScript and ZIP contents; this review does not remove or rewrite that metadata.

## Rendering and workflow review

The added empty-selection prompt renders fixed, repository-controlled translations. Locale strings select a translation; they are not interpolated as HTML. Both new translation files were checked for executable HTML, JavaScript URLs and credential-like values, with no suspicious entries found.

The PR adds a Node.js translation test to the existing validation workflow. It does not introduce a `pull_request_target` workflow or expose repository secrets to pull-request code. The repository's default workflow token permission is read-only. The existing release job requests `contents: write` only for uploading the release asset; it uses GitHub's job token rather than a committed personal token.

Upstream uses mutable action tags and a `main` container image. Pinning actions and container images to immutable references would be separate supply-chain hardening; those references were not introduced by this contribution.

## Validation

- Translation coverage, placeholders, locale overrides and new own-property regression tests pass.
- Build freshness and JavaScript syntax checks pass.
- The browser suite retains only the failures reproduced in untouched upstream; no additional browser failures or runtime errors were introduced by the resolver hardening.
- The test ZIP contains only the built JavaScript, installation notes, build metadata, browser results and checksums. Generated local packages are not tracked in Git.

Secret scanning can miss unknown formats and does not replace reviewing configuration before publishing. Live Home Assistant credentials and deployment configuration were outside this review's scope and were not read.
