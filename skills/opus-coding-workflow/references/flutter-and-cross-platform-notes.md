# Flutter & Cross-Platform Notes (Reference)

Load this file only when working on Flutter networking, native bridge plugins, or iOS IPA archive triage.

## Flutter Networking Domain Expansion
When adding a new networking domain enum case, inventory 5 points before patching:
1. Enum owner file (usually in `core/networking` package, not the feature call-site).
2. Grouped `switch` cases sharing init behavior (headers, interceptors, pinning, parser).
3. Bootstrap HTTP client / `Dio` registration list.
4. App config baseUrl getter or intentional reuse of an existing base URL.
5. Call-sites/usecases migrating from old domain to new domain (missing client registration fails at runtime).

## Flutter Native-Bridge Endpoint Discovery
- Start from Dart config/init params (`baseUrl`, `token_provider`, `custom_header`, `network_config`, `pinning_certs`) and example/README integration.
- Separate output into 3 layers: (1) config keys & injection sites, (2) concrete base URLs in integration/example, (3) native-bridge method names (distinct from REST endpoint paths).
- If REST paths live inside compiled native SDK/AAR/framework binaries, state that explicitly—never guess unverified paths like `/upload` or `/compare`. Group by capability and note auth/header fields (`jwt_token`, `token`, `refId`, `userId`) with file-path + line-range citations.

## Flutter iOS Archive Triage
- Run `fvm flutter build ipa --release --no-codesign` first to separate compile/archive errors from codesign/export errors.
- If it passes and `build/ios/archive/Runner.xcarchive` exists, report archive success clearly (missing `.ipa` is expected with `--no-codesign`).
- Verify real artifact metadata on disk: `.xcarchive` size, `CFBundleIdentifier`, `CFBundleShortVersionString`, `CFBundleVersion` from `Info.plist`.
