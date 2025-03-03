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



# ULyraExperienceManagerComponent

## 1. Class/Struct Name
**ULyraExperienceManagerComponent**  
> A `UGameStateComponent` that orchestrates the loading, activation, and unloading of a “Lyra Experience.” Implements `ILoadingProcessInterface` for loading screens.

---

## 2. Overview
`ULyraExperienceManagerComponent` manages the lifecycle of a given `ULyraExperienceDefinition`, including loading assets and game feature plugins, executing any associated `GameFeatureAction`s, and broadcasting events when the experience is fully operational. It also handles deactivation logic when a level or game session ends.

---

## 3. Key Responsibilities / Purpose
1. **Load and Configure “Experience” Data**:  
   - Loads the specified `ULyraExperienceDefinition` and its assets.
2. **Activate Game Features**:  
   - Detects which game feature plugins must be enabled for the current experience, and activates them.
3. **Execute/Deactivate Experience Actions**:  
   - Runs actions (e.g., registering gameplay tags, spawning custom subsystems) when the experience is activated.
   - Cleans up those actions during deactivation or end of play.
4. **Broadcast Experience State**:  
   - Notifies listeners (via delegates) once the experience is fully loaded (`OnExperienceLoaded*`).

---

## 4. Dependencies & Relationships
- **Inherits From**: 
  - `UGameStateComponent` (for lifecycle and replication within the game state).
  - `ILoadingProcessInterface` (to determine loading screen display logic).
- **References / Uses**:
  - `ULyraExperienceDefinition`: The asset defining which actions and plugins the experience needs.
  - `ULyraAssetManager`: For asynchronously loading assets.
  - `UGameFeaturesSubsystem`: For activating game feature plugins.
  - `ULyraExperienceActionSet` and `UGameFeatureAction`: Contain the actual operations performed during activation/deactivation.

---

## 5. Important Members

### Enums
- **`ELyraExperienceLoadState`**  
  - Tracks the internal state of the component’s load process.  
  - Values include: `Unloaded`, `Loading`, `LoadingGameFeatures`, `LoadingChaosTestingDelay`, `ExecutingActions`, `Loaded`, `Deactivating`.

### Variables
1. **`TObjectPtr<const ULyraExperienceDefinition> CurrentExperience`** *(ReplicatedUsing = OnRep_CurrentExperience)*  
   - Reference to the loaded experience definition.  
   - Remains `nullptr` until set, then triggers the load sequence.

2. **`ELyraExperienceLoadState LoadState`**  
   - Tracks the current stage of loading or unloading. Updated as tasks complete.

3. **`TArray<FString> GameFeaturePluginURLs`**  
   - List of plugin URLs required by the experience. Populated and activated during loading.

4. **`int32 NumGameFeaturePluginsLoading`**  
   - Counter for how many plugin load operations are currently in progress.

5. **`FOnLyraExperienceLoaded OnExperienceLoaded_HighPriority, OnExperienceLoaded, OnExperienceLoaded_LowPriority`**  
   - Delegates that fire when the experience completes loading.  
   - Split into three priority tiers so high-priority systems can process first.

### Methods
1. **`SetCurrentExperience(FPrimaryAssetId ExperienceId)`**  
   - Kicks off the loading process for a given experience asset ID.  
   - Immediately loads the definition from the asset path, checks it, and sets `CurrentExperience`.

2. **`CallOrRegister_OnExperienceLoaded_HighPriority / OnExperienceLoaded / OnExperienceLoaded_LowPriority`**  
   - Registers or immediately calls delegates that wait for the experience to finish loading.

3. **`GetCurrentExperienceChecked() const`**  
   - Returns `CurrentExperience` if `LoadState` is `Loaded`.  
   - Asserts if called too early (i.e., before the experience is fully loaded).

4. **`IsExperienceLoaded() const`**  
   - Returns `true` if `LoadState` is `Loaded` and `CurrentExperience` is non-null.

5. **`OnRep_CurrentExperience()`** *(UFUNCTION)*  
   - RepNotify function triggered when `CurrentExperience` changes.  
   - Calls `StartExperienceLoad()` to begin load sequence on clients.

6. **`StartExperienceLoad()`**  
   - Internal function that initiates asynchronous loading of the experience assets and sets up the plugin activation steps.

7. **`OnExperienceLoadComplete()`**  
   - Called when all required assets have finished loading. Proceeds to load/activate the required game feature plugins.

8. **`OnGameFeaturePluginLoadComplete(const UE::GameFeatures::FResult&)`**  
   - Called each time a single game feature plugin finishes loading/activation.  
   - When all plugins are done, calls `OnExperienceFullLoadCompleted`.

9. **`OnExperienceFullLoadCompleted()`**  
   - Finalizes the experience load (execution of actions, broadcasting delegates).  
   - Also includes an optional “chaos testing delay” for artificially testing load times.

10. **`EndPlay(const EEndPlayReason::Type EndPlayReason) override`**  
    - Cleans up or deactivates any loaded plugins, calling deactivation methods on relevant actions.

11. **`ShouldShowLoadingScreen(FString& OutReason) const override`**  
    - Part of `ILoadingProcessInterface`. Returns `true` if the experience is not yet fully loaded.

---

## 6. Implementation Notes & Lifecycle
1. **Setting the Experience**  
   - `SetCurrentExperience` is typically called by the game mode or another system with a valid `FPrimaryAssetId`.
2. **Loading Assets & Plugins**  
   - Once `CurrentExperience` is assigned, the component transitions through states (`Loading` → `LoadingGameFeatures` → optional chaos delay → `ExecutingActions` → `Loaded`).
   - Relevant game features are activated using `UGameFeaturesSubsystem::LoadAndActivateGameFeaturePlugin`.
3. **Chaos Testing Delay**  
   - Configured via console variables (`lyra.chaos.ExperienceDelayLoad.MinSecs` and `.RandomSecs`). Injects random delays to mimic network or data load variability.
4. **Actions Execution**  
   - Experience-based `GameFeatureAction`s are executed in the `ExecutingActions` stage. This can include blueprint or code logic hooking into gameplay systems.
5. **Unload / EndPlay**  
   - `EndPlay` triggers deactivation of all loaded game feature plugins and experience actions (though partial support for asynchronous deactivation is noted as incomplete).
6. **Delegate Order**  
   - Experience load completion fires three delegates in a specific order:
     1. `OnExperienceLoaded_HighPriority`
     2. `OnExperienceLoaded`
     3. `OnExperienceLoaded_LowPriority`

---

## 7. Example Usage

```cpp
// In your GameState or GameMode:

void AMyCustomGameState::LoadMyChosenExperience()
{
    // Suppose we have an asset ID for the chosen experience
    FPrimaryAssetId MyExperienceAssetId("LyraExperienceDefinition", "MyCoolExperience");

    ULyraExperienceManagerComponent* ExpManager = FindComponentByClass<ULyraExperienceManagerComponent>();
    if (ExpManager)
    {
        ExpManager->CallOrRegister_OnExperienceLoaded_HighPriority(
            FOnLyraExperienceLoaded::FDelegate::CreateLambda([](const ULyraExperienceDefinition* LoadedExperience)
            {
                UE_LOG(LogTemp, Log, TEXT("Experience loaded (high priority)!"));
                // Perform any immediate setup or state changes you want.
            }));

        ExpManager->CallOrRegister_OnExperienceLoaded(
            FOnLyraExperienceLoaded::FDelegate::CreateLambda([](const ULyraExperienceDefinition* LoadedExperience)
            {
                UE_LOG(LogTemp, Log, TEXT("Experience loaded (normal priority)!"));
                // Normal post-load logic goes here.
            }));

        ExpManager->SetCurrentExperience(MyExperienceAssetId);
    }
}
```
In practice, the game mode or another controller typically calls `SetCurrentExperience` to specify which experience to load when the match starts. Other systems can respond by binding delegates.

---

### 8. Common Pitfalls & Edge Cases

- **Calling `GetCurrentExperienceChecked` Too Early**  
  This will `check` (assert) if called before the experience finishes loading. Use `CallOrRegister_OnExperienceLoaded` to defer logic until safe.

- **Forgetting Plugin Deactivation**  
  By default, plugins stay active once loaded. `EndPlay` attempts to deactivate them, but partial or asynchronous deactivation is currently limited.

- **Chaotic Testing Delays**  
  The random delay for testing can cause confusion if not aware; can appear as an unexpected hitch in loading.

- **Asynchronous Deactivation**  
  The code logs that advanced or partial deactivation is not fully supported. Breaking changes or errors could arise if your actions rely heavily on async teardown.

---

### 9. Future Improvements or TODOs

- **Robust Error Handling**  
  The current logic `check`s on failures, but might need graceful fallback for production scenarios.

- **Partial / Ordered Unloading**  
  The system “leaks” game feature plugins if multiple experiences are loaded sequentially. A more robust approach would unload only the unneeded plugins.

- **Asynchronous Deactivation**  
  The code has a partial design for asynchronous action teardown but lacks full implementation.

---

### 10. FAQs / Troubleshooting

**Q**: Why is my experience never transitioning to “Loaded”?  
**A**: Ensure that the asset references and plugin names in `ULyraExperienceDefinition` are valid. Invalid plugin URLs or missing assets can stall the process.

**Q**: Can I load multiple experiences concurrently?  
**A**: The component is designed around a single `CurrentExperience`. Loading multiple experiences at once would require custom logic or a separate manager for each.

**Q**: What if I want to disable the chaos load delay?  
**A**: Set console variables `lyra.chaos.ExperienceDelayLoad.MinSecs` and `lyra.chaos.ExperienceDelayLoad.RandomSecs` to 0.

**Q**: Is there a blueprint-friendly way to listen for experience load completion?  
**A**: Use `CallOrRegister_OnExperienceLoaded` in C++ for best results, or create a blueprint event binding to that delegate. Alternatively, you can override `OnExperienceFullLoadCompleted` in a Blueprint subclass if needed.



# ULyraExperienceManager

## 1. Class/Struct Name
**ULyraExperienceManager**  
> An engine subsystem for managing experience-related game feature plugins, particularly in the Unreal Editor when multiple PIE sessions are running.

---

## 2. Overview
`ULyraExperienceManager` is an `UEngineSubsystem` used to track and manage game feature plugin activation requests in editor builds. It provides a simple reference-count mechanism, ensuring that if multiple PIE (Play-In-Editor) sessions request the same plugin, it remains active until all sessions release it. This helps avoid prematurely unloading a plugin that’s still in use by another PIE session.

---

## 3. Key Responsibilities / Purpose
- **Plugin Activation Arbitration**: Tracks how many systems or PIE sessions have requested activation of a given plugin.
- **Deferred Deactivation**: Prevents a plugin from being deactivated until all requesting sessions have released it.
- **PIE Session Reset**: Clears its internal tracking when a new PIE session begins.

---

## 4. Dependencies & Relationships
- **Inherits From**: `UEngineSubsystem` (meaning it’s globally available while the engine is running).
- **Relies On**:  
  - `GEngine->GetEngineSubsystem<ULyraExperienceManager>()` for global access.  
  - Editor-only (`WITH_EDITOR`) logic ensures that plugin reference counting only happens in editor builds.

---

## 5. Important Members

### Variables
- **`TMap<FString, int32> GameFeaturePluginRequestCountMap`**  
  - Maps a plugin URL (string) to how many times it has been requested.  
  - Increments on each activation request; decrements when a release is requested.

### Methods
1. **`void OnPlayInEditorBegun()`** *(Editor-Only)*  
   - Empties the `GameFeaturePluginRequestCountMap` at the start of a PIE session.  
   - Ensures leftover data from previous sessions doesn’t interfere with new requests.

2. **`static void NotifyOfPluginActivation(const FString PluginURL)`** *(Editor-Only)*  
   - Called when a system (e.g., `ULyraExperienceManagerComponent`) wants to activate a game feature plugin.  
   - Increments the reference count for that plugin URL in `GameFeaturePluginRequestCountMap`.

3. **`static bool RequestToDeactivatePlugin(const FString PluginURL)`** *(Editor-Only)*  
   - Called when a system wants to deactivate a game feature plugin.  
   - Decrements the reference count and only returns `true` (allowing the plugin to deactivate) if the count has reached zero.  
   - If the plugin is still in use (count > 0), returns `false` to prevent deactivation.

4. **Non-Editor Implementations**  
   - In non-editor builds, the methods do nothing or simply return a trivial response (`true`).

---

## 6. Implementation Notes & Lifecycle
- **Subsystem Initialization**:  
  - Created as an engine subsystem, `ULyraExperienceManager` is typically available once the engine is up.  
  - Accessed via `GEngine->GetEngineSubsystem<ULyraExperienceManager>()`.
- **Editor-Only Reference Counting**:  
  - The plugin management code (`NotifyOfPluginActivation`, `RequestToDeactivatePlugin`) is wrapped in `#if WITH_EDITOR`.  
  - Ensures that the feature is only active during PIE sessions and doesn’t affect packaged builds.

---

## 7. Example Usage
```cpp
#if WITH_EDITOR
void SomeEditorFunction()
{
    // Example plugin URL
    FString PluginURL = TEXT("MyGameFeaturePluginURL");

    // Acquire usage
    ULyraExperienceManager::NotifyOfPluginActivation(PluginURL);

    // ...

    // Later, release usage
    bool bCanDeactivate = ULyraExperienceManager::RequestToDeactivatePlugin(PluginURL);
    if (bCanDeactivate)
    {
        // Actually deactivate the plugin now
    }
}
#endif
```
In actual practice, these methods are typically called under the hood by other Lyra classes like `ULyraExperienceManagerComponent`. Direct usage in game code is usually minimal.

---

### 8. Common Pitfalls & Edge Cases

- **Non-Editor Environments**  
  The reference counting is disabled (does nothing) in a non-editor build. If you rely on it at runtime, your logic won’t be invoked.

- **Multiple PIE Sessions**  
  Make sure to call `OnPlayInEditorBegun()` when starting a new PIE session so that stale references are not carried over.

- **Missing Plugin URL**  
  If the plugin URL doesn’t match what `UGameFeaturesSubsystem` expects, reference counting might be incremented but never decremented.

---

### 9. Future Improvements or TODOs

- **Support Non-Editor Environments**  
  If desired, the logic could be extended to manage game feature plugin usage in live or shipping builds.

- **More Robust Error Handling**  
  Currently, the methods use asserts and checks. Production scenarios might require graceful handling if a plugin URL is invalid or not found.

---

### 10. FAQs / Troubleshooting

**Q**: Can I use `NotifyOfPluginActivation` in a packaged game build?  
**A**: The reference counting code is excluded when `WITH_EDITOR` is not defined, so it effectively won’t do anything in production builds.

**Q**: Does `OnPlayInEditorBegun()` automatically clear all references?  
**A**: Yes, it calls `GameFeaturePluginRequestCountMap.Empty()` to reset everything at the start of a PIE session.

**Q**: How do these static methods get called in the larger Lyra codebase?  
**A**: Typically, `ULyraExperienceManagerComponent` or other classes that load game feature plugins call `NotifyOfPluginActivation` and `RequestToDeactivatePlugin` to handle plugin lifecycle in editor builds.



# ULyraExperienceDefinition

## 1. Class/Struct Name
**ULyraExperienceDefinition**  
> A `UPrimaryDataAsset` that describes an entire “Lyra Experience.” It defines which plugins to activate, a default pawn, and a collection of “actions” (game feature actions or action sets) to execute for that experience.

---

## 2. Overview
`ULyraExperienceDefinition` is central to how Lyra structures “experiences.” An experience typically encapsulates a set of features, gameplay elements, and default data (e.g., pawn class) required for a particular game mode or scenario. This asset can:
- Reference additional game features that should be activated (via plugin URLs).
- Provide a default pawn data to ensure players have the correct character setup.
- Aggregate multiple `UGameFeatureAction`s and `ULyraExperienceActionSet`s to perform specific tasks on load or unload.

---

## 3. Key Responsibilities / Purpose
- **Define a Set of Game Features**: Lists the plugin names/URLs that should be enabled for this experience.
- **Specify the Default Pawn Data**: Establishes which pawn data class (e.g., `ULyraPawnData`) is used when no custom pawn data is specified.
- **Load/Unload Actions**: Contains `UGameFeatureAction`s and child `ULyraExperienceActionSet`s that define actions to perform throughout the game’s lifecycle.

---

## 4. Dependencies & Relationships
- **Inherits From**: `UPrimaryDataAsset` — enabling asynchronous loading and grouping of content.
- **References**:
  - `UGameFeatureAction`: Defines logic for loading, registering, and unloading features.
  - `ULyraExperienceActionSet`: Bundles actions and can further reference additional plugins.

This asset is typically used by the `ULyraExperienceManagerComponent`, which triggers loading and unloading of the defined features at runtime.

---

## 5. Important Members

1. **`TArray<FString> GameFeaturesToEnable`**  
   - List of plugin names/URLs that this experience requires.  
   - The manager will load and activate these plugins so the experience works as intended.

2. **`TObjectPtr<const ULyraPawnData> DefaultPawnData`**  
   - A pointer to the default pawn data.  
   - Provides a baseline pawn class and associated attributes if no custom pawn data is specified in other gameplay logic.

3. **`TArray<TObjectPtr<UGameFeatureAction>> Actions`**  
   - Instanced actions that execute code or blueprint logic upon load/unload.  
   - Each action can add functionality such as registering gameplay tags, spawning managers, or handling UI setup.

4. **`TArray<TObjectPtr<ULyraExperienceActionSet>> ActionSets`**  
   - An array of additional `ULyraExperienceActionSet` assets that group multiple actions.  
   - Encourages composition rather than deep inheritance.

---

## 6. Implementation Notes & Lifecycle
- **Data Validation**  
  - Overrides `IsDataValid` to ensure that each action is valid. If there’s a null entry, it’s flagged as invalid.  
  - Ensures that deep blueprint inheritance (e.g., blueprint subclasses of blueprint subclasses) is not currently supported; the system encourages composition instead.
- **Asset Bundles**  
  - Overrides `UpdateAssetBundleData` to incorporate any extra assets from `UGameFeatureAction`s.  
  - Helps with efficient loading/unloading in the `ULyraAssetManager`.
- **Editor vs. Runtime**  
  - At design time, you set up `ULyraExperienceDefinition` via the editor, referencing the required plugins and actions.  
  - At runtime, something like `ULyraExperienceManagerComponent` will set this as the current experience and begin the load process.

---

## 7. Example Usage

```cpp
// Pseudocode: Setting up a ULyraExperienceDefinition object in the editor

ULyraExperienceDefinition* MyExperience = NewObject<ULyraExperienceDefinition>();
MyExperience->GameFeaturesToEnable.Add("SomePluginURL");
MyExperience->DefaultPawnData = MyPawnDataAsset; // e.g., a reference to a ULyraPawnData

// Suppose we have some GameFeatureAction that spawns a special UI
UGameFeatureAction* ActionUI = NewObject<UGameFeatureAction>();
MyExperience->Actions.Add(ActionUI);

// Or we can add an ActionSet that references multiple actions
MyExperience->ActionSets.Add(MyActionSetReference);

// Once this is saved and assigned to an FPrimaryAssetId, the ULyraExperienceManagerComponent
// can load it, enabling the plugin and triggering these actions at runtime.
```
In practice, these references are usually established in the Unreal Editor UI rather than in code.

---

### 8. Common Pitfalls & Edge Cases

- **Null Actions**  
  If an entry in `Actions` is null, data validation fails.

- **Blueprint Inheritance**  
  The class checks if it has multiple blueprint layers. If so, the data validation warns to prefer composition via `ActionSets` instead of deep blueprint subclassing.

- **Incorrect Plugin Names**  
  Typos in `GameFeaturesToEnable` will cause the plugin load step to fail silently or produce warnings, potentially halting the experience load process.

---

### 9. Future Improvements or TODOs

- **Support for Partial Overriding**  
  Currently, you can’t “override” only certain aspects of an experience without duplicating the entire asset or using `ActionSets`.

- **Additional Validation**  
  More robust checks for plugin URLs or asset references to catch errors earlier.

- **Enhanced Editor UI**  
  Tools to easily link `GameFeatureActions` and preview their impact could improve user experience.

---

### 10. FAQs / Troubleshooting

**Q**: Can I load multiple experiences at once?  
**A**: Typically, only one experience is active at a time in Lyra. If you need more, you must manage the merges or load multiple definitions with custom logic.

**Q**: Why does my blueprint-subclassed `ULyraExperienceDefinition` give a warning?  
**A**: The system discourages multi-level blueprint inheritance. Check for a blueprint parent class that is already a subclass of `ULyraExperienceDefinition` and consider using `ActionSets` instead.

**Q**: What happens if `DefaultPawnData` is null?  
**A**: Lyra logic may fall back to a global default pawn data. However, this can lead to unexpected behavior if the experience expects a specific pawn class.

**Q**: Why won’t my new plugin load when the experience starts?  
**A**: Ensure the plugin name in `GameFeaturesToEnable` matches exactly what `UGameFeaturesSubsystem` expects. Typos or missing plugin definitions will prevent loading.
