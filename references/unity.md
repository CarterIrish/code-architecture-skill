# Unity Architecture Reference

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

## Pattern Choices

These are architectural decisions where multiple approaches are equally valid in Unity. Present the options during the interview and let the user choose. Each choice is labeled with a short name.

### Inter-System Communication

Present when the project has 3+ systems that need to talk to each other (e.g., combat → UI, player → audio, inventory → crafting).

**A) Direct references** — Systems hold serialized references to each other (dragged in the Inspector or found via `GetComponent`). Simple, visible in the editor, easy to debug.

```csharp
public class PlayerHealth : MonoBehaviour {
    [SerializeField] private HealthBarUI healthBar;
    public void TakeDamage(int amount) { hp -= amount; healthBar.UpdateDisplay(hp); }
}
```

Best for: Small projects, tightly coupled systems that always exist together, users who prefer explicit connections.

**B) C# events / delegates** — Systems fire events; listeners subscribe. Decoupled — the sender doesn't know who's listening.

```csharp
public class PlayerHealth : MonoBehaviour {
    public static event Action<int> OnHealthChanged;
    public void TakeDamage(int amount) { hp -= amount; OnHealthChanged?.Invoke(hp); }
}
// HealthBarUI subscribes: PlayerHealth.OnHealthChanged += UpdateDisplay;
```

Best for: Medium projects, systems that shouldn't know about each other, users comfortable with delegates.

**C) ScriptableObject event channels** — Events defined as SO assets. Fully decoupled — systems reference a shared SO asset instead of each other. The "Ryan Hipple pattern."

```csharp
[CreateAssetMenu] public class GameEvent : ScriptableObject {
    private List<GameEventListener> listeners = new();
    public void Raise() { foreach (var l in listeners) l.OnEventRaised(); }
}
```

Best for: Larger projects, designer-friendly workflows (events visible in the Inspector as assets), maximum decoupling. Higher setup cost than options A or B.

### State Machine Style

Present when the project has entities with multiple behavior states (player, enemies, game flow).

**A) Enum + switch** — A state enum and a switch statement in Update. Minimal code, easy to follow.

```csharp
enum EnemyState { Patrol, Chase, Attack }
void Update() {
    switch (currentState) {
        case EnemyState.Patrol: Patrol(); break;
        case EnemyState.Chase: Chase(); break;
    }
}
```

Best for: Entities with <5 states, simple transitions, portfolio projects where clarity matters more than scalability.

**B) State pattern (class per state)** — Each state is a class with Enter/Execute/Exit methods. Clean separation, easy to add new states without touching existing ones.

```csharp
public abstract class State { public virtual void Enter() {} public virtual void Execute() {} public virtual void Exit() {} }
public class PatrolState : State { public override void Execute() { /* patrol logic */ } }
```

Best for: Entities with 5+ states, complex transition logic, projects that will grow. More boilerplate upfront.

**C) Animator-driven** — States defined in Unity's Animator Controller, with StateMachineBehaviour scripts on each state. Transitions managed visually.
Best for: States that are tightly coupled to animations (character controllers, boss fights). Not great for logic-only state machines (game flow, UI screens).

### Data Architecture

Present when the project has tunable values (stats, items, difficulty, config).

**A) ScriptableObjects** — Data defined as SO assets, editable in the Inspector. The recommended Unity pattern for data-driven design.
Best for: Teams, designers who need to tweak values without code changes, systems with many data variants (items, enemies, abilities).

**B) JSON / XML config files** — Data stored as text files in Resources or StreamingAssets. Loaded at runtime.
Best for: Users who prefer editing data in a text editor, projects where data might be modded or loaded from external sources, users coming from non-Unity backgrounds.

**C) Hardcoded constants** — Values defined directly in code as `const` or `static readonly` fields.
Best for: Very small projects, game jam scope, values that truly never change. Don't use this for anything you'll want to tune later — the iteration speed penalty is real.

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

## Procedural Generation Patterns

### Tile-Based / Grid Generation

The most accessible approach for 2D games. The world is a grid of cells, each assigned a tile type.

- **Random fill + Cellular Automata** — Fill grid randomly, then run CA smoothing passes. Produces organic cave-like layouts. Simple to implement, easy to tune (birth/death thresholds, iteration count).
- **BSP (Binary Space Partitioning)** — Recursively subdivide a rectangle into rooms, then connect them with corridors. Produces structured dungeon layouts with distinct rooms. Good for roguelikes.
- **Drunkard's Walk** — Random agent carves paths through a filled grid. Produces winding, natural corridors. Combine multiple walkers for variety.
- **Wave Function Collapse (WFC)** — Constraint-based: define which tiles can be adjacent, then collapse the grid respecting those rules. Produces complex, visually coherent layouts. Higher implementation complexity — save this for when simpler methods can't achieve the desired aesthetic.

### Room-Based Generation (Roguelikes)

Common flow: generate rooms → place rooms (no overlap) → connect with corridors → populate with content.

- Separate the generation pipeline into stages: layout → connectivity → population → validation.
- Store the generated map as data (a 2D array of cell types) before instantiating GameObjects. This makes it easy to validate, serialize, and debug.
- Use a seed value (`Random.InitState(seed)`) so levels are reproducible for debugging and sharing.

### Scaling Advice

- **Start with the simplest algorithm that produces playable results.** Random fill + CA or a basic room placer is enough for a prototype. Don't start with WFC.
- **Build the generation pipeline so algorithms are swappable.** Define an interface like `ILevelGenerator` that returns a `TileMap` or `int[,]`. The rest of the game doesn't care how the level was made.
- **Test generation in isolation.** Build a debug view that renders the raw grid data (colored squares) without loading sprites, enemies, or game systems. This lets you iterate on generation parameters 10x faster.

## Time & Cycle Systems

### Day/Night Cycle

Core components:

- **WorldClock** — Tracks in-game time as a float (0.0 = midnight, 0.5 = noon, 1.0 = next midnight). Ticks forward each frame scaled by a time multiplier. Expose the current time, day count, and a normalized "time of day" value.
- **LightingController** — Listens to WorldClock, interpolates between lighting presets (dawn, day, dusk, night). Use a Gradient or AnimationCurve for smooth color transitions. For 2D: a full-screen overlay with a multiply blend mode is the simplest approach.
- **CycleEventBroadcaster** — Fires events at specific times: `OnDawn`, `OnDusk`, `OnNightStart`, `OnMorning`. Other systems subscribe to these rather than polling the clock.

### NPC Schedules / World State Changes

- Define schedules as data (ScriptableObject or JSON): `{ npcId, time, location, activity }`.
- Schedules react to CycleEventBroadcaster events, not to raw time checks. This keeps NPCs decoupled from the clock implementation.
- For "shops close at night" — the shop system subscribes to `OnNightStart`/`OnMorning` events, toggles an `isOpen` flag. The UI checks the flag. Don't hardcode time checks in the shop script.

### Scaling Advice

- **Start with a simple float counter and a few threshold events.** You don't need a full calendar system for a game with a day/night cycle. A normalized 0-1 float with 4 event thresholds (dawn, midday, dusk, midnight) covers most needs.
- **Separate visual time from gameplay time.** The lighting transition and the "shops close" mechanic should be driven by the same clock but not dependent on each other. This lets you tune visual pacing independently of gameplay pacing.

## Save System Patterns

### Approach Selection

- **PlayerPrefs** — Built-in, key-value only. Fine for settings (volume, resolution). Not for game state — it's not designed for structured data, has size limits, and is stored in platform-specific locations.
- **JSON Serialization** (JsonUtility or Newtonsoft.Json) — Serialize game state to a JSON file. Good for most indie projects. Human-readable, easy to debug, adequate performance for saves under ~10MB.
- **Binary Serialization** (BinaryFormatter is deprecated; use custom binary or MessagePack) — Faster, smaller files. Use when JSON is too slow or save files are large. Less debuggable.
- **ScriptableObject snapshots** — Serialize SO runtime data. Works well if your game state already lives in SOs.

### Save Architecture

```
SaveManager (singleton or static)
├── Collects ISaveable data from all registered systems
├── Serializes to SaveData struct/class
├── Writes to disk (Application.persistentDataPath)
└── Handles save slots, auto-save timing

ISaveable interface
├── GetSaveData() → returns serializable data
└── LoadSaveData(data) → restores state from data
```

Key pattern: Systems implement `ISaveable` and register with the SaveManager. On save, the manager iterates through registered systems, collects their data, bundles it into a single `SaveData` object, and writes to disk. On load, reverse the process.

### What to Save vs. Reconstruct

- **Save**: Player position, inventory contents, quest progress, world state flags, day count, RNG seed.
- **Reconstruct**: Procedurally generated levels (save the seed, regenerate the level), enemy positions if they respawn, UI state, cached calculations.
- Rule of thumb: save the minimum state needed to reconstruct the player's experience. If you can regenerate it deterministically, store the seed instead of the result.

## Audio Architecture

### Basic Structure

- **AudioManager** (singleton) — Owns audio sources, controls master volume, handles music transitions.
- **Sound Library** (ScriptableObject) — Map sound IDs to AudioClips with default volume/pitch/spatial settings. Systems request sounds by ID, not by clip reference.
- **Separate music and SFX channels** — Independent volume controls, different spatial behavior (music is 2D, SFX can be 3D).

### Common Patterns

- **Object pooling for SFX** — Don't instantiate an AudioSource per sound. Maintain a pool of AudioSource components, check one out, play, return it.
- **Music crossfading** — Two AudioSources for music. Fade one out while fading the other in. Use coroutines or DOTween for the transition.
- **Event-driven audio** — Systems fire events ("player_damaged", "item_pickup"), AudioManager subscribes and plays the corresponding sound. This keeps audio logic out of gameplay scripts.

### Scaling Advice

- **Start with a simple static AudioManager** that plays clips directly. A full sound library SO + pooling system is a nice-to-have for polish, not a requirement for a prototype.
- **Don't build a music system until you have music.** A single `AudioSource.PlayOneShot()` call in the AudioManager is fine until you need crossfading or dynamic layering.
