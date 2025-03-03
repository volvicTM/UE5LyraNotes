# UE5LyraNotes

# ALyraGameMode

### 1. Class/Struct Name
**ALyraGameMode** – The primary game mode class for the Lyra project, responsible for player initialization, pawn selection, experience loading, and match flow.

---

### 2. Overview
`ALyraGameMode` is the core game mode in the Lyra sample project. It manages how players and bots enter the game, what pawn (character) they control, and how the in-game “Experience” (game rules, data, etc.) is loaded. This class serves as the backbone of the gameplay loop, ensuring that players are spawned with the correct pawn data and that the overall match flow is properly coordinated.

---

### 3. Key Responsibilities / Purpose
- **Experience Management**: Loads and applies the `ULyraExperienceDefinition` to set up game-specific rules and data.  
- **Player/Bot Spawning**: Chooses the correct spawn points and pawn classes for players and AI controllers.  
- **Game Flow Initialization**: Handles initialization sequences (e.g., `InitGame`, `HandleStartingNewPlayer`) so that the game transitions smoothly into a playable match.  
- **Match Assignment**: Determines which “Experience” the map should use (e.g., from command line, developer settings, or world settings).  
- **Player Restart Logic**: Manages how and when players or bots are restarted (respawned) after death or travel.

---

### 4. Dependencies & Relationships
- **Inherits From**: `AModularGameModeBase` (which extends `AGameModeBase` in Unreal).
- **Depends On**:
  - `ALyraGameState`: The custom game state that holds match-wide data and components.
  - `ULyraExperienceManagerComponent`: Attached to the `ALyraGameState`; it loads and manages the current experience data.
  - `ULyraPawnData`: Describes which pawn (character class) to use and other pawn-related data.
  - `ULyraPlayerSpawningManagerComponent`: Handles spawn logic and spawn points.
  - `ULyraAssetManager`: Used to load assets (such as experiences and default pawn data).
  - `ALyraPlayerController`, `ALyraPlayerBotController`: The different controllers for players and AI bots.
  - Various other Lyra classes (e.g., `ALyraPlayerState`, `ALyraCharacter`, `ALyraHUD`) that define state, visuals, or additional gameplay functionality.

All of these classes/structs interact in a typical UE5 flow: the `GameMode` talks to the `GameState` and specialized manager components to spawn controllers and pawns properly.

---

### 5. Important Members

#### Delegate
- **FOnLyraGameModePlayerInitialized OnGameModePlayerInitialized**  
  A multicast delegate broadcast when a player (or bot) is fully initialized. Useful for hooking additional setup logic after a player logs in.

#### Functions
1. **`const ULyraPawnData* GetPawnDataForController(const AController* InController) const`**  
   - Determines which `ULyraPawnData` to use for the specified controller (player or AI).  
   - Looks first at the controller’s `ALyraPlayerState` to find existing pawn data.  
   - Falls back to the default data specified by the loaded `ULyraExperienceDefinition` or the global default from `ULyraAssetManager`.

2. **`virtual void InitGame(const FString& MapName, const FString& Options, FString& ErrorMessage) override`**  
   - UE lifecycle method called when the map is being initialized.  
   - Schedules a timer to handle the “Match Assignment” next frame (so that any command line or developer settings are fully read).

3. **`void HandleMatchAssignmentIfNotExpectingOne()`**  
   - Determines which “Experience” to use for the match by checking command line arguments, developer settings, etc.  
   - Calls `OnMatchAssignmentGiven` once the experience is chosen (or defaults to a fallback).

4. **`void OnMatchAssignmentGiven(FPrimaryAssetId ExperienceId, const FString& ExperienceIdSource)`**  
   - Called once an experience has been identified.  
   - Tells the `ULyraExperienceManagerComponent` to load that specific experience asset.

5. **`void OnExperienceLoaded(const ULyraExperienceDefinition* CurrentExperience)`**  
   - Runs after the experience data is fully loaded.  
   - Spawns any already-connected players who were waiting for the experience to finish loading.

6. **`bool IsExperienceLoaded() const`**  
   - Quick check to see if the `ULyraExperienceManagerComponent` has finished loading the current experience.

7. **`virtual UClass* GetDefaultPawnClassForController_Implementation(AController* InController) override`**  
   - Chooses the default pawn class, typically `ALyraCharacter` or something specified by `ULyraPawnData`.  
   - Overridden to integrate with custom pawn data logic.

8. **`virtual APawn* SpawnDefaultPawnAtTransform_Implementation(AController* NewPlayer, const FTransform& SpawnTransform) override`**  
   - Spawns a pawn for a given controller at the specified transform.  
   - Ensures the `ULyraPawnExtensionComponent` is set with the correct `PawnData`.

9. **`virtual bool ShouldSpawnAtStartSpot(AController* Player) override`**  
   - Overrides default Unreal behavior to always use `PlayerSpawningManagerComponent`, ignoring the standard player start spot logic.

10. **`virtual void HandleStartingNewPlayer_Implementation(APlayerController* NewPlayer) override`**  
    - Called when a new player joins. Defers the actual start if the experience isn’t loaded yet.

11. **`virtual AActor* ChoosePlayerStart_Implementation(AController* Player) override`**  
    - Delegates the selection of a spawn point to the `ULyraPlayerSpawningManagerComponent`.

12. **`virtual void FinishRestartPlayer(AController* NewPlayer, const FRotator& StartRotation) override`**  
    - Performs final steps after a player’s respawn. Calls into `ULyraPlayerSpawningManagerComponent` to finalize the process.

13. **`virtual bool PlayerCanRestart_Implementation(APlayerController* Player) override`**  
    - Forwards to `ControllerCanRestart` to check if the player is allowed to respawn.

14. **`virtual bool ControllerCanRestart(AController* Controller)`**  
    - Unified logic for both players and bots to determine if a restart is allowed.

15. **`virtual void InitGameState() override`**  
    - Standard UE lifecycle method. Hooks into the experience manager’s load event.

16. **`virtual void GenericPlayerInitialization(AController* NewPlayer) override`**  
    - Calls the base implementation, then broadcasts `OnGameModePlayerInitialized` once the player is fully ready.

17. **`UFUNCTION(BlueprintCallable) void RequestPlayerRestartNextFrame(AController* Controller, bool bForceReset = false)`**  
    - Schedules a controller restart on the next frame. Useful for timed respawns or quick re-initialization.

18. **`virtual bool UpdatePlayerStartSpot(AController* Player, const FString& Portal, FString& OutErrorMessage) override`**  
    - No-op here; the real spawning logic is deferred until after full initialization.

19. **`virtual void FailedToRestartPlayer(AController* NewPlayer) override`**  
    - Called when a restart attempt fails. Tries again on the next frame if conditions allow.

#### Variables
- Custom classes like `GameStateClass`, `PlayerControllerClass`, `PlayerStateClass`, `DefaultPawnClass`, `HUDClass` are set in the constructor to specific Lyra classes (e.g., `ALyraGameState`, `ALyraPlayerController`).

---

### 6. Implementation Notes & Lifecycle
- **Construction**  
  Uses `FObjectInitializer` in `ALyraGameMode::ALyraGameMode` to set default classes (e.g., `ALyraGameState::StaticClass()`).

- **UE5 Lifecycle Methods**  
  - `InitGame` is called when the level is loaded. The game mode sets a timer to pick or confirm the correct “Experience.”  
  - `InitGameState` is then called, registering the callback for when the experience actually finishes loading.  
  - `HandleStartingNewPlayer` is only invoked once the experience data is ready (to avoid spawning pawns with missing data).  
  - On each successful or failed attempt at spawning a pawn, methods like `FinishRestartPlayer`, `FailedToRestartPlayer`, and `ShouldSpawnAtStartSpot` shape the normal UE spawn flow.

- **Experience Flow**  
  - The core is: choose or detect a `ULyraExperienceDefinition`, load it via `ULyraExperienceManagerComponent`, then spawn players.

- **Deferred Operations**  
  - Several calls (`RequestPlayerRestartNextFrame`, `HandleMatchAssignmentIfNotExpectingOne`) rely on timers or next-frame callbacks to ensure certain game data is ready or to avoid race conditions.

---

### 7. Example Usage

```cpp
// In Project Settings -> Maps & Modes -> GameMode, set the default to ALyraGameMode
// or in a level blueprint:

ALyraGameMode* CurrentGameMode = GetWorld()->GetAuthGameMode<ALyraGameMode>();
if (CurrentGameMode)
{
    // Example: Force a player restart next frame
    APlayerController* PC = /* retrieve player controller somehow */;
    CurrentGameMode->RequestPlayerRestartNextFrame(PC, /*bForceReset=*/true);
}

// Typical usage occurs automatically; you just set ALyraGameMode as your game mode
// and rely on the built-in spawn logic and experience assignment.
```
In many cases, you’d configure `ALyraGameMode` in your project `.ini` files or level overrides, and the rest is handled by the Lyra framework.

---

### 8. Common Pitfalls & Edge Cases
- **Experience Not Loaded**: If you call `RestartPlayer` or spawn logic before the `ULyraExperienceDefinition` is fully loaded, pawns might spawn with incomplete data. The game mode defers spawning until it’s safe.  
- **Missing Pawn Data**: If `GetPawnDataForController` returns `nullptr` (e.g., no default pawn data found), spawns can fail. The logs will indicate the error.  
- **Forgetting to Call `Super`**: When overriding `HandleStartingNewPlayer`, `FinishRestartPlayer`, or similar, ensure you call `Super` to preserve vital logic.  
- **Dedicated Server Setup**: The code includes logic for dedicated servers (e.g., `TryDedicatedServerLogin`, `HostDedicatedServerMatch`). Skipping or removing that might break server-based tests if you rely on that flow.

---

### 9. Future Improvements or TODOs
- **Customizable Experience Logic**: The experience selection process is somewhat “hard-coded” to command line, developer settings, or world settings. A more flexible, data-driven approach might be desired.  
- **Refine Dedicated Server Flow**: The login/hosting flow could be expanded for different online subsystems or advanced matchmaking needs.  
- **Better Error Handling**: Currently logs errors if an asset can’t be found or spawned. Could be extended to provide UI feedback or fail gracefully.

---

### 10. FAQs / Troubleshooting

**Q**: Why is my player not spawning immediately after I start the game?  
**A**: The `Experience` might still be loading. `ALyraGameMode` waits for the `ULyraExperienceManagerComponent` to finish before spawning. Check your logs for messages indicating the loading status.

**Q**: How do I override the default pawn class in Lyra?  
**A**: Either provide a custom `ULyraPawnData` with a different pawn class or override `GetPawnDataForController` to select your own logic.

**Q**: I tried customizing the spawn location logic, but it’s not working. Why?  
**A**: Lyra uses a `ULyraPlayerSpawningManagerComponent` that overrides standard Unreal spawn points. Ensure you modify or subclass that component for custom spawn behaviors.

**Q**: Is the “dedicated server” code mandatory?  
**A**: It’s there for advanced or large-scale multiplayer scenarios. You can remove or ignore it if you’re not using dedicated servers, but keep in mind it may break references within the code if you rely on them.



# ALyraGameState

## 1. Class/Struct Name
**ALyraGameState**  
> The base game state class used in the Lyra project. It implements `IAbilitySystemInterface` and tracks game-wide data such as server FPS and replay-recording state.

---

## 2. Overview
`ALyraGameState` is a derived `AModularGameStateBase` designed to manage global game logic, hold a shared `ULyraAbilitySystemComponent`, and coordinate gameplay experiences within the Lyra framework. It maintains replication of server information like FPS, handles replay-specific logic, and broadcasts gameplay “verb” messages to all connected clients.

---

## 3. Key Responsibilities / Purpose
- **Global Ability System**: Provides a centralized `ULyraAbilitySystemComponent` for game-wide effects and cues.
- **Server Metrics**: Tracks and replicates server FPS to clients.
- **Replay Recording**: Identifies which `APlayerState` is recording a replay and broadcasts changes.
- **Global Messaging**: Sends replicated “verb messages” to clients (both reliable and unreliable).

---

## 4. Dependencies & Relationships
- **Inherits From**: `AModularGameStateBase` (part of the Lyra modular framework) and implements `IAbilitySystemInterface`.
- **Key Class Dependencies**:
  - `ULyraAbilitySystemComponent`: The centralized ability system component for this game state.
  - `ULyraExperienceManagerComponent`: Manages the current gameplay experience (also attached to the game state).
  - `APlayerState`: Used to track players; includes references for replay features.
  - `UGameplayMessageSubsystem`: Handles broadcasting game messages to clients.

---

## 5. Important Members

### Variables
1. **`TObjectPtr<ULyraAbilitySystemComponent> AbilitySystemComponent`**  
   - A replicated ability system component responsible for handling game-wide abilities and cues.  
   - Initialized in the constructor and set to `Replicated` with `Mixed` replication mode.

2. **`TObjectPtr<ULyraExperienceManagerComponent> ExperienceManagerComponent`**  
   - Manages loading and applying the current gameplay “Experience.”  
   - Created in the constructor via `CreateDefaultSubobject`.

3. **`float ServerFPS`** *(Replicated)*  
   - Stores the current server frames per second.  
   - Updated every tick on the server and replicated to clients.

4. **`TObjectPtr<APlayerState> RecorderPlayerState`** *(Transient, ReplicatedUsing = OnRep_RecorderPlayerState)*  
   - Points to the `APlayerState` that recorded the replay.  
   - Only replicated in replay contexts; used to track the correct pawn for replay viewers.

5. **`FOnRecorderPlayerStateChanged OnRecorderPlayerStateChangedEvent`**  
   - A multicast delegate invoked whenever `RecorderPlayerState` changes.

### Methods
1. **`ALyraGameState(const FObjectInitializer&)`**  
   - Constructor setting up default subobjects (`AbilitySystemComponent`, `ExperienceManagerComponent`) and enabling ticking.

2. **Overrides of `AActor` / `AGameStateBase`:**  
   - `PreInitializeComponents()`, `PostInitializeComponents()`, `EndPlay()`, `Tick(float DeltaSeconds)`, `AddPlayerState(APlayerState*)`, `RemovePlayerState(APlayerState*)`, `SeamlessTravelTransitionCheckpoint(bool)`.  
   - Notable is `Tick` where `ServerFPS` is updated on the server side each frame.

3. **`virtual UAbilitySystemComponent* GetAbilitySystemComponent() const override;`**  
   - Required by `IAbilitySystemInterface`. Returns `AbilitySystemComponent`.

4. **`void MulticastMessageToClients(const FLyraVerbMessage Message)`** *(Unreliable)*  
   - Sends an “unreliable” verb message to all clients, typically for events that can tolerate potential packet loss (e.g. elimination announcements).

5. **`void MulticastReliableMessageToClients(const FLyraVerbMessage Message)`** *(Reliable)*  
   - Sends a “reliable” verb message to all clients, guaranteeing delivery. Recommended for critical notifications.

6. **`float GetServerFPS() const`**  
   - Returns the replicated server FPS.

7. **`void SetRecorderPlayerState(APlayerState* NewPlayerState)`**  
   - Identifies the `PlayerState` that recorded the replay. Called once per game session.

8. **`APlayerState* GetRecorderPlayerState() const`**  
   - Retrieves the replay-recording player state, if valid.

9. **`void OnRep_RecorderPlayerState()`**  
   - Replication callback for `RecorderPlayerState`. Broadcasts `OnRecorderPlayerStateChangedEvent`.

---

## 6. Implementation Notes & Lifecycle
- **Constructor**  
  - Initializes `AbilitySystemComponent` with replication enabled and a `Mixed` effect replication mode.  
  - Creates an `ULyraExperienceManagerComponent` for experience handling.
- **Pre/Post Initialization**  
  - `PostInitializeComponents` initializes the ability actor info (`InitAbilityActorInfo`) to link the GameState as both owner and avatar.
- **Tick**  
  - Each server tick updates `ServerFPS` from `GAverageFPS`.
- **Player State Management**  
  - Overrides `AddPlayerState` and `RemovePlayerState` to manage `PlayerArray`; also handles bot or inactive states during seamless travel transitions.
- **Message Broadcasting**  
  - Uses `MulticastMessageToClients_Implementation` for unreliably broadcasting a `FLyraVerbMessage`.  
  - `MulticastReliableMessageToClients_Implementation` calls the same method but marked “Reliable” to ensure delivery.

---

## 7. Example Usage

```cpp
// Typical usage is automatic: ALyraGameState is set as your GameStateClass in ALyraGameMode.
// Access the ability system or server FPS from other parts of your code:

ALyraGameState* LyraGS = GetWorld()->GetGameState<ALyraGameState>();
if (LyraGS)
{
    float CurrentServerFPS = LyraGS->GetServerFPS();
    ULyraAbilitySystemComponent* ASC = LyraGS->GetLyraAbilitySystemComponent();

    // Example: Broadcasting a quick message
    FLyraVerbMessage Message;
    Message.Verb = FGameplayTag::RequestGameplayTag(FName("GameEvent.Example"));
    Message.Instigator = ASC;
    // ... fill out other fields as needed ...
    
    LyraGS->MulticastReliableMessageToClients(Message);
}
```

### 8. Common Pitfalls & Edge Cases
- **Forgetting to Initialize the Ability System**: Ensure `InitAbilityActorInfo` is set up properly in `PostInitializeComponents`.
- **ServerFPS Update**: Only updated on servers. If your environment is not truly server-authoritative, `ServerFPS` might remain 0 on clients.
- **RecorderPlayerState Replication**: Marked `COND_ReplayOnly`; if you expect it to replicate outside replay, you need a different approach.
- **Multicast vs. Reliable Multicast**: Use the reliable version (`MulticastReliableMessageToClients`) for critical messages that cannot be dropped.

---

### 9. Future Improvements or TODOs
- **Ability System Refactoring**: Some games might want a separate ability system for certain world-level effects vs. purely “global” effects.
- **Enhanced Replay Features**: Additional logic for dynamic selection of the replay’s `RecorderPlayerState`.
- **Extended Experience Manager**: The `ExperienceManagerComponent` could be leveraged more deeply for game state transitions.

---

### 10. FAQs / Troubleshooting

**Q**: Why can’t I see `ServerFPS` on clients?  
**A**: Make sure your network role is set to `ROLE_Authority` on the server, and check the replication is working. Non-dedicated sessions or local play can yield different behaviors.

**Q**: How do I use the `AbilitySystemComponent` from `ALyraGameState`?  
**A**: Access it via `GetLyraAbilitySystemComponent()` or `GetAbilitySystemComponent()`. Be sure to initialize abilities in `PostInitializeComponents` or later.

**Q**: Can I store references in `RecorderPlayerState` for non-replay usage?  
**A**: By default, it’s intended for replay logic only. If you require extra game logic, consider replicating a separate variable or using standard `PlayerState` patterns.

**Q**: Which messaging function should I use if reliability is uncertain?  
**A**: For ephemeral events (like chat messages or kill feeds), `MulticastMessageToClients` (unreliable) is fine. For important notifications (like game-winning conditions), use `MulticastReliableMessageToClients`.
