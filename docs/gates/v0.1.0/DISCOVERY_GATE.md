# Discovery Gate — Selo
Version: v0.1.0

## Project Classification
Type: Ecosystem Library
Deliverables:
- `selo-engine` — internal Gradle module; shades Cobalt behind `io.selo.internal.engine`; never published as a standalone artifact
- `selo-core` — pure JVM library; public API (`SeloRuntime`, `SeloScript`, `SeloSandbox`, `SeloValue`); zero MC imports; published to Maven Central
- `selo-fabric` — thin Fabric 1.21.1 bridge; wires MC lifecycle events into `selo-core`; published to Maven Central and Modrinth

## Concept Summary
Selo is a Lua scripting engine for Minecraft mod authors. It exposes a clean,
versioned Java API for embedding Lua scripts into mods — binding Java objects,
firing events, and enforcing a sandbox — with no Minecraft dependency in the
core library. The engine is backed by a shaded Cobalt runtime during the PoC
phase, fully abstracted behind Selo types so the implementation can be replaced
without any change to the public API. The long-term path is a clean Lua
implementation built on Java 21 virtual threads, replacing Cobalt entirely from
within `selo-engine`. No Cobalt type ever appears in a public signature.

## Scope
In:
- `selo-engine` internal module shading Cobalt into `io.selo.internal.engine`
- `SeloRuntime` — creates and manages a Lua execution context
- `SeloScript` — represents a loaded, executable Lua script
- `SeloSandbox` — configures allowed stdlib modules; blocks `io`, `os`, `require` by default
- `SeloValue` — Selo's public representation of a Lua value; no Cobalt types exposed
- Java-to-Lua object binding via `SeloRuntime.bind(String name, Object obj)`
- Event dispatch from Java into Lua via `SeloRuntime.dispatch(String event, Object... args)`
- `selo-fabric` mod entrypoint initializing `SeloRuntime` on world load
- Script loading from `config/selo/` at runtime
- PoC acceptance criteria (see below)

Out:
- `selo-neoforge` bridge
- `selo-cli` tooling
- Hot-reload of scripts at runtime
- Addon packaging or distribution system
- UX / HUD mod (separate project, separate repo)
- Fork of Cobalt (post-PoC path)
- Clean Lua implementation (long-term path)
- Party frames, raid frames, or any UI rendering

## PoC Acceptance Criteria
1. `selo-core` loads and executes a Lua script in a plain JVM unit test with no Minecraft dependency on the classpath
2. A Java object bound via `SeloRuntime.bind()` is callable from Lua and returns a correct value to the script
3. `SeloSandbox` blocks access to `io`, `os`, and `require` by default; a script attempting access MUST receive a Lua error, not a JVM exception
4. `selo-fabric` initializes a `SeloRuntime` on world load and executes a script loaded from `config/selo/`
5. Script output from criterion 4 appears in the Minecraft console without error or crash

## Engine Abstraction Constraint
`selo-engine` MUST NOT export any Cobalt type in any public or protected
signature. `selo-core` MUST NOT reference any class under the
`io.selo.internal.engine` package in its public API. All Lua values crossing
the `selo-core` public boundary MUST be represented as `SeloValue`. This
constraint MUST be enforced by Gradle module visibility configuration, not
discipline alone.

## Engine Replacement Path
Phase 1 — PoC: Cobalt shaded into `selo-engine` via Gradle shadow plugin.
Phase 2 — Fork: Cobalt source forked into `selo-engine` directly; shaded
           dependency removed; `selo-core` public API unchanged.
Phase 3 — Clean implementation: `selo-engine` rewritten against Java 21
           virtual threads; `selo-core` public API unchanged.

## Platform Targets
| Platform | Version | Priority |
|----------|---------|----------|
| Fabric   | 1.21.1  | P0       |
| Java     | 21      | P0       |
| NeoForge | TBD     | Deferred |

## SME Skills Required
| Skill | Status | Phase Needed |
|-------|--------|--------------|
| mc-mod-implementation | Installed | Implementation |
| mc-mod-assets | Installed | Implementation |
| mc-mod-community | Installed | Release |

## Open Questions
None — concept is fully defined.

## Repo
URL: https://github.com/johnverheek/selo-ui
Branch: feature/v0.1.0-discovery
