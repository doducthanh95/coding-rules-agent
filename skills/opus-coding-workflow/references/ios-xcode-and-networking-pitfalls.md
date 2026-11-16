# iOS Xcode & Networking Pitfalls (Reference)

Load this file only when debugging or modifying iOS / Xcode / Swift networking projects.

## Xcode `.xcodeproj` & Target Wiring
- Creating a `.swift` file on disk does **not** add it to the Xcode target. For `Cannot find 'TypeName' in scope`, check `project.pbxproj` (`PBXFileReference`, `PBXBuildFile`, group `children`, `PBXSourcesBuildPhase`).
- Add compile-time dependencies (e.g., interceptors, protocols) alongside new models/files.
- Never add two same-named types from legacy and new modules (e.g. `BaseRequestModel`) to the same target (`invalid redeclaration`). When two copies exist at different paths, verify which one is in `PBXSourcesBuildPhase` before editing.
- For `Multiple commands produce` or duplicate `*.stringsdata`, check sibling project embeds and same-basename source files in the same target.
- On workspace-based iOS projects, build with `.xcworkspace` (not `.xcodeproj`) so sibling frameworks resolve.
- Capture full build log first (`xcodebuild ... 2>&1 | tee /tmp/build.log`) before filtering with `rg -n "error:|warning:|BUILD (SUCCEEDED|FAILED)"`.
- For bulk `cannot find ... in scope` errors around a subsystem, verify disk files + `PBXSourcesBuildPhase` + README drift before patching individual types.

## Swift Types, Access Control & Decodable
- Cross-module subclassing requires `public`/`open` `init` on the superclass.
- Inheritance clause: superclass must appear before protocols (`class AppResponse<T: Decodable>: CoreBaseResponse, Decodable`).
- Non-final `Decodable` subclasses: mark `init(from:)` as `required`; call `try super.init(from: decoder)` if superclass is `Decodable`; avoid redundant `, Decodable`; do not shadow superclass stored properties like `message`.
- Default arguments of `public init` cannot reference `internal` symbols.
- `Result.failure` always requires a payload (`Result.failure(error)`).
- Keep UI stubs compatible with `IPHONEOS_DEPLOYMENT_TARGET` (e.g. avoid `UIButton.Configuration` on iOS 14).

## Networking & Monorepo Type Disambiguation
- Monorepo shared names (`BaseResponse.swift`, `BaseError.swift`, `BaseResponseModel.swift`): run `find` + `grep` across all modules to list every definition (class vs struct) and confirm target path with the user before patching.
- Contract drift in demo/type-erased wrappers: fix at the boundary adapter (e.g. wrap `BaseResponseModel<Data>` -> `AnyBaseResponse`) rather than rewriting all call sites; normalize internal errors (`CoreBaseError -> Error`) at public `completion` boundaries; check `init(code:message:)` availability on `CoreBaseError`.
- Token-refresh queue: ensure HTTP client surfaces 401 as an error with `statusCode` (not success-data); safely cast `err as? CoreBaseError` before reading error codes; verify end-to-end with deterministic fixture (e.g. `https://httpbin.org/bearer`).
- When updating shared contracts (`DomainNetwork.init`, `ApiHelper.request<R>`), grep all sibling modules in the workspace and remove dead mapper fields/params once unused.
