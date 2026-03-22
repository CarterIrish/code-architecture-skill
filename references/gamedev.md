# Game Development Architecture Reference

## Common Architectural Patterns

### Component-Based Architecture (Unity Default)
Unity's core pattern — attach MonoBehaviour components to GameObjects to compose behavior.
- Each component has a single responsibility.
- Communication via direct references, events, or a mediator.
- Use `GetComponent<T>()` sparingly at runtime; cache references in `Awake()` or `Start()`.

Best for: Most Unity projects. It's the path of least resistance and what the engine is built around.

### Manager / Singleton Pattern
Central manager objects that coordinate game-wide systems:
- **GameManager** — Game state, flow control (menus, gameplay, pause, game over).
- **AudioManager** — Sound playback, music transitions.
- **UIManager** — Screen transitions, HUD updates.
- **SaveManager** — Persistence, serialization.

Implementation: Use a base Singleton<T> MonoBehaviour or a static instance pattern. Be cautious with lazy initialization across scenes.

Pitfall: Overusing singletons creates hidden dependencies. If everything talks to GameManager, you've built a god object. Prefer events or dependency injection for loose coupling.

### ScriptableObject Data Architecture
Use ScriptableObjects as shared data containers and configuration:
- **Item/weapon/enemy definitions** — Data-driven design, editable in the Inspector.
- **Event channels** — ScriptableObject-based events for decoupled communication (Ryan Hipple's pattern).
- **Runtime sets** — Track active objects without singletons.
- **Configuration** — Tuning values, difficulty curves, loot tables.

Best for: Any project with designer-tunable data, inventory systems, status effects, crafting recipes.

### Entity Component System (ECS / DOTS)
Data-oriented design for performance-critical scenarios:
- **Entities** — Just an ID.
- **Components** — Pure data structs (no logic).
- **Systems** — Process all entities with matching component sets.

Best for: Large-scale simulations, thousands of entities, performance-critical projects. Overkill for most indie/student projects.

### State Machines
Essential for managing entity behavior:
- **Player states** — Idle, Running, Jumping, Attacking, Stunned.
- **Enemy AI states** — Patrol, Chase, Attack, Flee.
- **Game flow states** — MainMenu, Loading, Playing, Paused, GameOver.

Implementation options:
- Simple enum + switch (fine for small state counts).
- State pattern with a base State class and transitions.
- Animator-driven state machines for animation-coupled behavior.

### Observer / Event Pattern
Decouple systems that need to react to each other:
- C# events and delegates for in-code communication.
- UnityEvent for Inspector-wired callbacks.
- ScriptableObject event channels for fully decoupled cross-system events.
- Use when: UI needs to react to health changes, audio needs to react to combat, achievements need to track everything.

## Project Structure (Unity)

```
Assets/
├── Scripts/
│   ├── Core/              # GameManager, state machines, singletons
│   ├── Player/            # Player controller, input, abilities
│   ├── Enemies/           # Enemy AI, spawning, behavior
│   ├── Systems/           # Inventory, crafting, save/load, dialogue
│   ├── UI/                # HUD, menus, screen management
│   ├── Data/              # ScriptableObject definitions
│   └── Utils/             # Extension methods, helpers, constants
├── Prefabs/
│   ├── Characters/
│   ├── Environment/
│   ├── UI/
│   └── VFX/
├── ScriptableObjects/     # Asset instances of SO definitions
├── Scenes/
├── Art/
│   ├── Sprites/ or Models/
│   ├── Materials/
│   ├── Animations/
│   └── Textures/
├── Audio/
│   ├── SFX/
│   └── Music/
└── Resources/             # Only for runtime-loaded assets (use sparingly)
```

## System Design Principles (Game-Specific)

### Data-Driven Design
Separate data from logic wherever possible:
- Define stats, items, and abilities as ScriptableObjects or JSON/XML.
- Systems read data at runtime; designers tune values without touching code.
- Makes balancing and iteration dramatically faster.

### Scene Management
- Use additive scene loading for modular level design.
- Keep a persistent "Boot" or "Core" scene for managers.
- Use SceneManager.LoadSceneAsync for seamless transitions.

### Input Handling
- Use Unity's new Input System for cross-platform support.
- Abstract input behind an interface so systems don't depend on input method directly.
- Map actions (Jump, Attack, Interact) rather than raw keys.

## Common Pitfalls (Game Dev)

- **God objects**: GameManager doing everything. Split responsibilities into focused managers.
- **Update() soup**: Too much logic in Update(). Use events, coroutines, or state machines to reduce per-frame checks.
- **String-based references**: `GameObject.Find("Player")` and tag comparisons are fragile. Use direct references, dependency injection, or ScriptableObject channels.
- **Premature optimization with ECS/DOTS**: Unless you demonstrably need the performance, standard MonoBehaviour architecture is simpler, faster to develop, and easier to debug.
- **No separation between game logic and presentation**: Keep combat math separate from VFX/animation triggers. This makes testing and balancing much easier.
- **Ignoring assembly definitions**: For larger projects, use .asmdef files to speed up compilation and enforce module boundaries.
