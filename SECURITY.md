# Security and public publishing

This is a cleaned source snapshot, not a guarantee of production security.

## Changes in this package

- Real Supabase project configuration replaced with placeholders; no configured `.env` included.
- Build caches, compiled web/native output, IDE state, local SDK paths and generated Capacitor assets excluded.
- Native Android and iOS template source retained; Android widget bridge/receiver installed.
- Git ignores cover environment variants, signing keys, native build output and local service configuration.
- HTTPS/public-key configuration guard and localhost-only development server default added.
- Android backup disabled and cleartext traffic disallowed in the manifest and installer.
- Missing administrator pairing SQL restored with deliberately invalid placeholders.
- Source-pattern checks, GitHub CI and Dependabot configuration added.
- The xcode build dependency uses a scoped uuid 11.1.1 override. This addresses
  https://github.com/advisories/GHSA-w5hq-g745-h8pq while keeping CommonJS support.
  xcode's UUID-generation path was smoke-tested; a full signed iOS build was not.

## Before publishing

1. Copy `.env.example` to `.env` locally and configure only your public client values. Never put secrets in VITE variables: the bundler exposes them even when the runtime guard rejects the configuration.
2. Run `npm run security:check`, `npm test`, `npm run build`, and `npm audit`.
3. Review `git diff --cached` before pushing. CI detects problems after a push; it cannot undo a disclosure. Enable repository secret scanning/push protection where available.
4. This ZIP has no Git history. If publishing an existing repository, separately scan its full history with a history-aware tool such as Gitleaks. Replacing working files does not remove past commits. Rotate any actual secret discovered there.
5. Do not publish actual administrator pairing SQL or personal user data. The included pair.sql intentionally fails until its placeholders are replaced; keep configured copies outside the repository.

## Deployment boundaries

The schema enables RLS with owner-only writes and explicit viewer membership. It was reviewed as source, not exercised against your live database in this pass. Validate owner A, paired viewer B, unpaired user C and anonymous access using their actual sessions before relying on a deployment. No live policies, accounts or service settings were changed.

Sessions use the starter's standard Supabase client storage. Widget content is cached on the device and visible on the home screen. Disabling Android backup reduces one exposure path; it is not encrypted session storage or a universal guarantee against vendor device-transfer behavior. Revocation cannot erase an offline copy. Do not use this starter for highly sensitive notes without a broader storage/privacy review.

The bundled source checker uses selected credential patterns, not comprehensive detection or history scanning. Dependency audits only cover known advisories in their registry at the time of checking. Native builds/signing and on-device behavior still need validation.

Report vulnerabilities privately to the repository maintainer; do not post credentials or personal data in a public issue.
