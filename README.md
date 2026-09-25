# Window Flow

> **Organize your desktop. Control application relationships. Isolate what needs to stay offline.**

Window Flow is a local-first desktop workspace and security manager that turns application windows into intentional, policy-aware workspaces.

Assign roles to windows, connect applications, save complete workflows, and move applications into a visual **Dead Zone** to isolate them from the network where the operating system supports reliable enforcement.

## Why Window Flow?

Modern desktops are built around applications, but not around the relationships between them.

A typical workflow might involve:

```text
Browser → Research → Notes → Editor → Export
```

Window Flow makes those relationships explicit.

Instead of managing isolated windows, you manage:

* **Windows** — what is open
* **Roles** — where information is intended to flow
* **Relationships** — which applications are connected
* **Workspaces** — complete saved workflows
* **Policies** — what applications are allowed to do
* **Security zones** — where applications can operate

---

## Core Concepts

### Window

Every managed application window becomes a Window Flow object.

A window can have:

* Application identity
* Process identity
* Position and size
* Role
* Workspace membership
* Relationships
* Security state

### Roles

Windows can be assigned a simple data-flow role:

| Role          | Meaning                                  |
| ------------- | ---------------------------------------- |
| `SOURCE`      | Primarily provides information           |
| `DESTINATION` | Primarily receives information           |
| `FLEXIBLE`    | Can both provide and receive information |

### Connections

Connections describe the intended relationship between applications.

```text
Browser → Notes
Notes → Editor
Editor ↔ File Manager
```

Connections are **logical relationships**.

They are separate from physical window positioning.

### Snap

Snap controls where a window physically sits.

```text
┌───────────────┬───────────────┐
│    SOURCE     │  DESTINATION  │
│    Browser    │     Notes     │
│               │               │
└───────────────┴───────────────┘
```

### Workspace

A Workspace is a saved desktop workflow.

A workspace can contain:

* Applications
* Window positions
* Window sizes
* Roles
* Connections
* Security policies
* Workspace mode

Example:

```text
Research Workspace

Browser       SOURCE
Notes         DESTINATION
Terminal      FLEXIBLE

Browser  ─────→ Notes
Terminal ─────→ Notes
```

Save it once. Restore the workflow later.

---

# Dead Zone

The **Dead Zone** is Window Flow's spatial security concept.

Drag an application window into the Dead Zone and Window Flow attempts to isolate the underlying application process from the network.

```text
┌──────────────────────────────────────────────┐
│                                              │
│               NORMAL DESKTOP                 │
│                                              │
│     Browser             Notes                │
│                                              │
│                                              │
│                  ┌───────────────┐           │
│                  │   DEAD ZONE   │           │
│                  │               │           │
│                  │   🔒 OFFLINE  │           │
│                  └───────────────┘           │
│                                              │
└──────────────────────────────────────────────┘
```

The intended flow is:

```text
Window
   ↓
Process identification
   ↓
Security policy
   ↓
Network isolation
   ↓
Verification
   ↓
ISOLATED
```

Window Flow must **never display an isolated state unless isolation has actually been verified**.

If isolation cannot be enforced or verified, the UI reports that limitation instead.

## Security States

```text
ONLINE
   ↓
ISOLATION_PENDING
   ↓
ISOLATED
   ↓
LOCKED_DOWN
```

Failure:

```text
ISOLATION_PENDING
        ↓
ISOLATION_FAILED
```

### Dead Zone

Intended behaviour:

* Network access blocked
* Isolation verified
* Controlled transfers where supported
* Explicit restoration required

### Lockdown

Lockdown is a stronger security state.

Where the platform permits:

* Network access blocked
* Clipboard transfers blocked
* Drag-and-drop blocked
* External transfer mechanisms restricted
* Security policy changes require explicit unlock

Security capabilities are platform-dependent.

Window Flow does not claim universal enforcement where the operating system does not provide the necessary controls.

---

# Capability States

Security features are reported using explicit capability states:

| State          | Meaning                                                   |
| -------------- | --------------------------------------------------------- |
| `ENFORCED`     | Policy is actively enforced                               |
| `DETECTED`     | Activity was detected but cannot necessarily be prevented |
| `UNDETERMINED` | State cannot currently be verified                        |
| `UNSUPPORTED`  | Platform does not provide the required capability         |

This distinction is fundamental to Window Flow.

**Detection is not enforcement.**

---

# Workspace Modes

### Normal

Standard desktop operation.

### Focus

Minimize distractions and expose only the active workflow.

### Privacy

Apply the workspace's configured privacy policies.

Example:

```text
Research Workspace
Mode: PRIVACY

Browser     SOURCE       ONLINE
Notes       DESTINATION  ONLINE
Sensitive   FLEXIBLE     ISOLATED
```

---

# Platform Strategy

Window Flow is designed around native operating-system capabilities rather than pretending every desktop behaves identically.

### Linux — Priority 1

Target:

* X11
* Wayland capability detection
* Window/process identification
* Window positioning
* Desktop overlays
* Network isolation

Wayland support will be capability-based. Window Flow will not assume X11-level control where Wayland does not expose it.

### macOS — Priority 2

Target:

* Accessibility APIs
* Window management
* Process identification
* Network filtering/isolation capabilities
* Security state verification

### Windows — Priority 3

Target:

* Win32
* UI Automation
* Process identification
* Firewall/network isolation
* Security state verification

---

# Architecture

The core architecture separates desktop orchestration from platform-specific implementation.

```text
                    ┌──────────────────┐
                    │    UI / Shell    │
                    └────────┬─────────┘
                             │
                    ┌────────▼─────────┐
                    │   Window Flow    │
                    │      Core        │
                    ├──────────────────┤
                    │ Windows          │
                    │ Workspaces       │
                    │ Relationships    │
                    │ Roles            │
                    │ Policies         │
                    │ Security State   │
                    └────────┬─────────┘
                             │
             ┌───────────────┼────────────────┐
             │               │                │
       ┌─────▼─────┐   ┌─────▼─────┐   ┌─────▼─────┐
       │   Linux   │   │   macOS   │   │  Windows  │
       │  Adapter  │   │  Adapter  │   │  Adapter  │
       └─────┬─────┘   └─────┬─────┘   └─────┬─────┘
             │               │                │
       ┌─────▼─────┐   ┌─────▼─────┐   ┌─────▼─────┐
       │ X11 /     │   │Accessibility│  │ Win32 /   │
       │ Wayland   │   │ / Network  │   │ UIA / WFP │
       └───────────┘   └────────────┘   └───────────┘
```

## Recommended Stack

### Core

**Rust**

Responsible for:

* Window model
* Workspace engine
* Relationship graph
* Policy engine
* Security state machine
* Persistence
* Platform abstraction

### UI

Platform-appropriate desktop UI layered over the Rust core.

### Storage

**SQLite**

Local-first persistence for:

* Workspaces
* Window metadata
* Roles
* Relationships
* Policies
* Preferences

No cloud dependency is required for the core product.

---

# Privacy Model

Window Flow is designed to operate locally.

The core engine should **not collect or store**:

* Clipboard contents
* Keystrokes
* Screenshots
* Passwords
* Document contents
* Browser contents
* Private application data

Window Flow needs to understand the desktop environment, not the contents of the user's work.

---

# MVP

The first usable prototype should prove the core thesis rather than attempt to implement every platform simultaneously.

### Phase 1 — Desktop Orchestration

* [ ] Linux window detection
* [ ] X11 support
* [ ] Wayland capability detection
* [ ] Window overlays
* [ ] Window roles
* [ ] Edge snapping
* [ ] Logical connections
* [ ] Workspace creation
* [ ] Workspace save/load
* [ ] Workspace switching
* [ ] Tray application

### Phase 2 — Dead Zone

* [ ] Dead Zone UI
* [ ] Process identification
* [ ] Network isolation provider
* [ ] Isolation policy
* [ ] Isolation verification
* [ ] Security state machine
* [ ] Restore Network action
* [ ] Global Kill Network control where supported

### Phase 3 — Lockdown

* [ ] Clipboard policy
* [ ] Drag/drop policy
* [ ] Lockdown mode
* [ ] Security profiles
* [ ] Policy verification
* [ ] Fail-safe policy persistence

### Phase 4 — Additional Platforms

* [ ] macOS
* [ ] Windows

---

# Security Principles

Window Flow follows several non-negotiable principles.

### 1. Never fake security

If a policy cannot be enforced, say so.

### 2. Verify before claiming

A UI state such as:

```text
🔒 ISOLATED
```

must correspond to a verified underlying security state.

### 3. Fail closed where practical

Security policies should remain active if the UI crashes or restarts, where the operating system permits independent enforcement.

### 4. Make limitations visible

Platform limitations are part of the product model, not hidden implementation details.

### 5. Local-first

The desktop security engine should not depend on a remote service.

---

# Repository Structure

A possible initial structure:

```text
window-flow/
├── README.md
├── LICENSE
├── Cargo.toml
│
├── crates/
│   ├── core/
│   │   ├── window/
│   │   ├── workspace/
│   │   ├── relationship/
│   │   ├── policy/
│   │   └── security/
│   │
│   ├── platform/
│   │   ├── linux/
│   │   ├── macos/
│   │   └── windows/
│   │
│   └── app/
│
├── ui/
│
├── migrations/
│
└── docs/
    ├── architecture.md
    ├── security.md
    └── platform-capabilities.md
```

---

# Development Philosophy

Window Flow should be built from **capabilities upward**, not assumptions downward.

The first question for every platform feature is:

> **Can the operating system reliably expose and enforce this state?**

If yes, implement it.

If it can only be detected, expose detection.

If it cannot be verified, report `UNDETERMINED`.

If it is unavailable, report `UNSUPPORTED`.

The product should remain useful even when security enforcement is unavailable.

---

# Current Thesis

Window Flow is not simply another window manager.

It is a **spatial desktop orchestration and security layer**.

Applications have:

**places, roles, relationships, workspaces and security states.**

```text
             WORKSPACE
                  │
       ┌──────────┼──────────┐
       │          │          │
    WINDOW     WINDOW     WINDOW
       │          │          │
     ROLE       ROLE       ROLE
       │          │          │
       └──── RELATIONSHIP ───┘
                  │
              POLICY
                  │
          SECURITY STATE
```

The long-term goal is simple:

> **Make the desktop an intentional environment rather than a collection of unrelated windows.**

---

# Status

**v0 — Architecture / Prototype**

The project is currently focused on validating:

1. Reliable window-to-process identification
2. Workspace orchestration
3. Logical application relationships
4. Linux network isolation
5. Verifiable Dead Zone behaviour

The most important technical proof-of-concept is:

> **Can Window Flow reliably identify the process behind a desktop window and place that process into a verifiable network-isolated state on Linux?**

If that primitive works reliably, the Dead Zone becomes a foundation for the broader Window Flow security model.

---

## License

License to be determined.
