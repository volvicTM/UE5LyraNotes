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



# ULyraExperienceActionSet

## 1. Class/Struct Name
**ULyraExperienceActionSet**  
> A `UPrimaryDataAsset` that groups together a set of `UGameFeatureAction` objects and optional game feature plugins to enable, typically as part of an overall “experience.”

---

## 2. Overview
`ULyraExperienceActionSet` is an asset designed to bundle multiple actions (and potential plugin dependencies) into a single collection. You can reference these action sets within a `ULyraExperienceDefinition` (or other assets) to compose gameplay functionality without needing deep inheritance. This facilitates modular design, letting you mix and match sets of actions to create different experiences.

---

## 3. Key Responsibilities / Purpose
- **Action Bundling**: Holds an array of `UGameFeatureAction` instances that should execute when loaded.  
- **Optional Plugin Dependencies**: Can specify a list of game feature plugin names/URLs to ensure certain functionalities are active.  
- **Composition Over Inheritance**: Encourages reusing sets of actions across multiple experiences without creating multiple blueprint subclasses.

---

## 4. Dependencies & Relationships
- **Inherits From**: `UPrimaryDataAsset`, enabling asynchronous loading and data bundling support.
- **References**:
  - `UGameFeatureAction`: The actions to perform (e.g., registering game tags, spawning managers, hooking up UI).  
  - Other code or assets can read `GameFeaturesToEnable` to load the required plugins.

`ULyraExperienceActionSet` is often referenced by `ULyraExperienceDefinition` but can be used anywhere action sets are beneficial.

---

## 5. Important Members

1. **`TArray<TObjectPtr<UGameFeatureAction>> Actions`**  
   - Holds the instanced actions that run when the asset is loaded.  
   - Typically includes logic for enabling UI elements, gameplay systems, or other per-experience features.

2. **`TArray<FString> GameFeaturesToEnable`**  
   - Additional game feature plugins (by name/URL) that the set depends on.  
   - Helps ensure these plugins are loaded so the relevant actions can execute without missing dependencies.

---

## 6. Implementation Notes & Lifecycle
- **Data Validation**  
  - The class checks each entry in `Actions`; if any are null, it marks the asset as invalid.  
  - Helps catch missing references early.
- **Editor-Only Asset Bundles**  
  - `UpdateAssetBundleData` lets the asset manager know about extra assets each `UGameFeatureAction` needs. This improves load/unload handling.
- **Composition**  
  - `ULyraExperienceActionSet` can be reused across multiple experiences by being added to the `ActionSets` array in `ULyraExperienceDefinition`. This approach avoids deep or repeated blueprint inheritance.

---

## 7. Example Usage

```cpp
// Example: Creating/setting up an ActionSet in C++ (though typically done in the Editor):

ULyraExperienceActionSet* MyActionSet = NewObject<ULyraExperienceActionSet>();
MyActionSet->GameFeaturesToEnable.Add("MyPluginURL");

// Add an action:
UGameFeatureAction* SomeAction = NewObject<UGameFeatureAction>();
MyActionSet->Actions.Add(SomeAction);

// Later, reference MyActionSet in a ULyraExperienceDefinition or other system:
ULyraExperienceDefinition* MyExperience = NewObject<ULyraExperienceDefinition>();
MyExperience->ActionSets.Add(MyActionSet);
```
Most configuration is done within the Unreal Editor by assigning actions in the asset details panel.

---

### 8. Common Pitfalls & Edge Cases

- **Null Actions**  
  If an entry in `Actions` is null, data validation fails and logs an error.

- **Mismatched Plugin Names**  
  Values in `GameFeaturesToEnable` must match what the `UGameFeaturesSubsystem` expects; typos or incorrect names prevent those plugins from loading.

- **Asset vs. Code**  
  While you can create and manage `ULyraExperienceActionSet` in code, it’s typical to define these sets as assets in the editor for easy reuse.

---

### 9. Future Improvements or TODOs

- **Advanced Dependency Management**  
  Could potentially track which `UGameFeatureAction` depends on which plugin, allowing partial loading/unloading per action.

- **Extended Validation**  
  Checking if plugin names in `GameFeaturesToEnable` actually exist or if actions are compatible might prevent runtime errors.

- **Blueprint Support**  
  Currently marked as `NotBlueprintable`; that’s intentional, but some teams might want to extend it with blueprint logic for custom use cases.

---

### 10. FAQs / Troubleshooting

**Q**: Why do I see an error about a null action entry?  
**A**: One of the elements in `Actions` is null. Remove or replace it with a valid `UGameFeatureAction` reference.

**Q**: Are `GameFeaturesToEnable` automatically loaded?  
**A**: Generally, an external system (e.g., `ULyraExperienceManagerComponent`) picks these up and handles the plugin loading sequence. Make sure you have the correct references or plugin names.

**Q**: Can I nest multiple action sets?  
**A**: Indirectly, yes — you can embed multiple `ULyraExperienceActionSet` assets in your `ULyraExperienceDefinition`, but you don’t typically embed an `ActionSet` inside another `ActionSet`.

**Q**: Do I need to handle load/unload code manually?  
**A**: Typically no; once referenced by a `ULyraExperienceDefinition` or a manager system, actions are executed at the correct time. You only need to ensure you’ve set up references properly.



# UAsyncAction_ExperienceReady

## 1. Class/Struct Name
**UAsyncAction_ExperienceReady**  
> A BlueprintAsyncAction that waits for the Lyra experience to finish loading before triggering an event.

---

## 2. Overview
`UAsyncAction_ExperienceReady` is an asynchronous Blueprint node that monitors the game’s `AGameStateBase` and `ULyraExperienceManagerComponent`. It notifies you (via `OnReady`) when the experience has successfully loaded. This is particularly useful for Blueprint scripts that need to wait until the game’s “experience” is fully set up before running initialization logic.

---

## 3. Key Responsibilities / Purpose
- **Async Waiting for Experience**: Ensures that gameplay scripts don’t run until the Lyra experience data is present and fully loaded.  
- **Blueprint Integration**: Provides a node (`WaitForExperienceReady`) that can be called from Blueprints to handle asynchronous loading cleanly.  
- **Minimal Overhead**: Automatically self-destructs (`SetReadyToDestroy`) once the event has broadcast, preventing lingering references.

---

## 4. Dependencies & Relationships
- **Subclass of**: `UBlueprintAsyncActionBase`  
  - Integrates with Blueprint for asynchronous, latent function calls.
- **Uses**:
  - `ULyraExperienceManagerComponent` to check if the experience is loaded.  
  - `World->GameStateSetEvent` to wait for the `GameState` if it’s not initially ready.

The `OnReady` multicast delegate is broadcast once the experience has finished loading.

---

## 5. Important Members

### Delegate
- **`FExperienceReadyAsyncDelegate OnReady`**  
  - A dynamic multicast delegate called once the experience is confirmed ready.

### Functions
1. **`static UAsyncAction_ExperienceReady* WaitForExperienceReady(UObject* WorldContextObject)`**  
   - Creates an instance of this async action.  
   - Internally stores a weak reference to the current `UWorld`.  
   - Returns the async action object, which you can bind to via Blueprints.

2. **`void Activate()`** *(override)*  
   - Called after the node is created in Blueprint.  
   - Checks if there’s a valid `AGameStateBase`; if not, waits for one via `World->GameStateSetEvent`.  
   - Once `AGameStateBase` is ready, the action listens to the `ULyraExperienceManagerComponent` to determine if the experience is already loaded or not.

### Internal Steps
- **Step1_HandleGameStateSet**: Called when the engine reports that a `GameState` is available.  
- **Step2_ListenToExperienceLoading**: Looks up `ULyraExperienceManagerComponent`. If the experience is already loaded, schedule a next-frame callback to broadcast readiness; otherwise, registers a delegate for `OnExperienceLoaded`.  
- **Step3_HandleExperienceLoaded**: Triggered when `OnExperienceLoaded` fires; then moves on to broadcasting.  
- **Step4_BroadcastReady**: Broadcasts `OnReady` to all Blueprint listeners, then marks the action ready to destroy.

---

## 6. Implementation Notes & Lifecycle
- **Creation**: Typically created in a Blueprint using `WaitForExperienceReady(WorldContext)`.  
- **Activation**: Once created, `Activate` checks for an existing `GameState`:
  - If no game state is found yet, it subscribes to `GameStateSetEvent`.
  - Once it has a `GameState`, it fetches the `ULyraExperienceManagerComponent`.  
- **Delay a Frame**: Even if the experience is loaded, the action waits one frame (`SetTimerForNextTick`) before broadcasting, ensuring that any dependent logic isn’t instantly triggered in the same frame.  
- **Self Destruction**: After broadcasting, calls `SetReadyToDestroy()` to free itself.

---

## 7. Example Usage

**Blueprint Example**  
1. In your Blueprint graph, right-click and search for **Wait for Experience Ready**.  
2. Connect the **Execute** pin from your event (e.g., `Event BeginPlay`) to **Wait for Experience Ready**.  
3. On the `OnReady` pin, add logic that relies on the loaded experience.

```cpp
// C++ snippet illustrating usage:
void UMyBlueprintFunctionLibrary::StartMyAsyncLogic(UObject* WorldContextObject)
{
    UAsyncAction_ExperienceReady* AsyncNode = UAsyncAction_ExperienceReady::WaitForExperienceReady(WorldContextObject);
    if (AsyncNode)
    {
        AsyncNode->OnReady.AddDynamic(this, &UMyBlueprintFunctionLibrary::HandleExperienceReady);
    }
}

void UMyBlueprintFunctionLibrary::HandleExperienceReady()
{
    // Safe to proceed now, experience is fully loaded
}
```

---

### 8. Common Pitfalls & Edge Cases

- **Null WorldContext**  
  If `WaitForExperienceReady` is called with an invalid or null context, the action returns `nullptr` and never triggers.

- **GameState Not Yet Spawned**  
  The node waits for `GameStateSetEvent`. Ensure your game state is properly configured in your `GameMode`.

- **Multiple Wait Nodes**  
  You can have multiple nodes waiting for the same event; each one will broadcast when ready. Make sure you don’t inadvertently create multiple waiting nodes in the same flow unless intended.

---

### 9. Future Improvements or TODOs

- **Optional Additional Delays**  
  The single-frame delay is minimal; advanced usage might want user-defined waiting times.

- **Error Handling**  
  If the experience fails to load for some reason (e.g., missing plugin), this action might never trigger. It could be extended to handle error states or timeouts.

- **Dynamic Instances**  
  In larger or more dynamic levels, you may want additional checks for partial loading scenarios.

---

### 10. FAQs / Troubleshooting

**Q**: Does this require a Lyra-style `GameState`?  
**A**: Yes, it specifically looks for `ULyraExperienceManagerComponent` on the `GameState`. If you’re using a different framework, it won’t function as intended.

**Q**: Why does my callback never fire?  
**A**: Check that your `GameStateClass` is set to a class containing `ULyraExperienceManagerComponent` and that your “experience” is actually being loaded. If the experience load is skipped, the event won’t broadcast.

**Q**: Can I manually call `OnReady.Broadcast()`?  
**A**: You shouldn’t. The internal flow triggers `OnReady` at the correct time. Overriding that can break the async node’s lifecycle.

**Q**: Will this node broadcast if I call it after the experience is already loaded?  
**A**: Yes — if the experience is loaded, it schedules the broadcast for the next tick, ensuring consistent behavior (and not firing immediately within the same frame).



# ULyraUserFacingExperienceDefinition

## 1. Class/Struct Name
**ULyraUserFacingExperienceDefinition**  
> A `UPrimaryDataAsset` defining user-facing details for a Lyra experience, such as map selection, extra URL options, session settings, and UI elements.

---

## 2. Overview
`ULyraUserFacingExperienceDefinition` allows you to configure and present a game “experience” to players. It contains metadata like the map to load, the underlying `LyraExperienceDefinition` to use, UI text and icons, and session hosting parameters (like max players). This makes it straightforward to create distinct “front-end” entries for experiences, each with unique visuals and session properties.

---

## 3. Key Responsibilities / Purpose
- **User-Facing Configuration**: Defines how the experience is displayed (title, icon, description) and whether it’s shown in the front-end experience list.
- **Session Hosting Settings**: Includes map asset, additional URL arguments, max player count, replay recording, etc.
- **Default/Preferred Experiences**: Identifies default experiences for quick play or priority in UI.

---

## 4. Dependencies & Relationships
- **Inherits From**: `UPrimaryDataAsset` (enables referencing this class as a data-driven asset).
- **References / Uses**:  
  - `UCommonSession_HostSessionRequest`: Used to create a host session request with the specified settings.  
  - `FPrimaryAssetId`: Points to both the map (`MapID`) and the main Lyra experience definition (`ExperienceID`).
  - `ULyraReplaySubsystem` and `UCommonSessionSubsystem`: For replay support and session creation logic.

---

## 5. Important Members

1. **`FPrimaryAssetId MapID`**  
   - Identifies the map asset to load.  
   - Must be tagged as `"Map"` in the asset registry.

2. **`FPrimaryAssetId ExperienceID`**  
   - Specifies which `LyraExperienceDefinition` to use.  
   - Must match the `LyraExperienceDefinition` asset type in the project.

3. **`TMap<FString, FString> ExtraArgs`**  
   - Key-value pairs for additional URL options (e.g., `?param=value`).

4. **UI Fields**  
   - `FText TileTitle`, `FText TileSubTitle`, `FText TileDescription`, `TObjectPtr<UTexture2D> TileIcon`  
   - Used for displaying this experience’s title, subtitle, descriptive text, and icon in the front-end UI.

5. **`TSoftClassPtr<UUserWidget> LoadingScreenWidget`**  
   - Optional widget class to show as a loading screen when transitioning to or from this experience.

6. **`bool bIsDefaultExperience`**  
   - If true, this experience is given priority for quick-play or other default behaviors.

7. **`bool bShowInFrontEnd`**  
   - If true, the experience is visible in the front-end experiences list (e.g., main menu selection).

8. **`bool bRecordReplay`**  
   - If true and the platform supports replays, a replay will be recorded during the session.

9. **`int32 MaxPlayerCount`**  
   - The maximum number of players that can join a session hosted with this experience.

10. **`UFUNCTION CreateHostingRequest(const UObject* WorldContextObject) const`**  
    - Generates a `UCommonSession_HostSessionRequest` prepopulated with the above settings (`MapID`, `ExtraArgs`, `MaxPlayerCount`, etc.).

---

## 6. Implementation Notes & Lifecycle
- **Data Asset Configuration**: Typically set in the Unreal Editor, where you assign map assets, text, icons, and the experience ID.
- **Session Creation**  
  - `CreateHostingRequest` is the main entry point when hosting a match using these user-facing settings.  
  - If the `UCommonSessionSubsystem` is available, the new session request is configured for the relevant online mode, presence, and optionally replay recording.
- **Replay Support**  
  - Conditioned on `ULyraReplaySubsystem::DoesPlatformSupportReplays()`; if supported, the code sets `DemoRec` to enable replay recording if `bRecordReplay` is `true`.
- **Priority / Default Experience**  
  - The `bIsDefaultExperience` flag might be used by the front-end to place this experience at the top of a playlist or quick-play option.
- **Asset ID Requirements**  
  - `MapID` and `ExperienceID` must be valid primary assets, typically assigned in the editor’s asset reference fields.

---

## 7. Example Usage

```cpp
// Example usage in some session hosting logic:
ULyraUserFacingExperienceDefinition* MyUserFacingExp = LoadObject<ULyraUserFacingExperienceDefinition>(nullptr, TEXT("/Game/Experiences/Exp_Foo.Exp_Foo"));
if (MyUserFacingExp)
{
    // Build a host session request:
    UCommonSession_HostSessionRequest* HostRequest = MyUserFacingExp->CreateHostingRequest(WorldContextObject);
    if (HostRequest)
    {
        // Now pass HostRequest to the session subsystem for hosting
        UCommonSessionSubsystem* SessionSubsystem = GameInstance->GetSubsystem<UCommonSessionSubsystem>();
        if (SessionSubsystem)
        {
            SessionSubsystem->HostSession(nullptr, HostRequest);
        }
    }
}
```
In Blueprint, you’d typically reference the `ULyraUserFacingExperienceDefinition` asset in UI code or a game mode object, then call `CreateHostingRequest` when the player selects “Play.”

---

### 8. Common Pitfalls & Edge Cases

- **Invalid `ExperienceID` or `MapID`**  
  If these asset IDs don’t point to real assets, the session might fail to load or produce errors at runtime.

- **Replay Incompatibility**  
  Setting `bRecordReplay` to true on platforms without replay support has no effect. Check `ULyraReplaySubsystem::DoesPlatformSupportReplays()`.

- **Session Subsystem Unavailable**  
  If `UCommonSessionSubsystem` isn’t present or fails to create the request, the function falls back to creating a standalone `UCommonSession_HostSessionRequest`.

---

### 9. Future Improvements or TODOs

- **Dynamic UI Updates**  
  Could add logic to dynamically adjust the text or icon based on current game state or external data.

- **Validation**  
  More robust checks for whether `MapID` and `ExperienceID` are valid, potentially warning the user if references are missing.

- **Extended Hosting Options**  
  Possibly add more configuration fields (voice chat, matchmaking regions, etc.) to tailor the session request further.

---

### 10. FAQs / Troubleshooting

**Q**: How do I ensure my map loads properly?  
**A**: Verify that `MapID` points to a valid map asset and that the map is marked as “Map” in the asset registry.

**Q**: Can I override the loading screen widget at runtime?  
**A**: You can set `LoadingScreenWidget` in the data asset, but to change it at runtime, you’d likely need a specialized approach (e.g., override in code or modify the data at runtime).

**Q**: Why is my experience not listed in the front-end?  
**A**: Check if `bShowInFrontEnd` is true. Also ensure your UI system or front-end logic enumerates and displays these experience assets.

**Q**: Do I need a dedicated server for these settings?  
**A**: Not necessarily. If you’re hosting a listen server, the request is still valid. Dedicated servers often override certain fields like `bUsePresence`.



# ALyraWorldSettings

## 1. Class/Struct Name
**ALyraWorldSettings**  
> A custom `AWorldSettings` subclass that specifies the default Lyra experience to load when a server opens the map.

---

## 2. Overview
`ALyraWorldSettings` serves as a specialized world settings class in the Lyra project. It allows map designers or developers to designate a “default gameplay experience” (i.e., a `ULyraExperienceDefinition`) that the server will use when loading this particular map, unless overridden by a user-facing experience selection. It can also force standalone mode in Play-In-Editor (PIE) scenarios for front-end or menu levels.

---

## 3. Key Responsibilities / Purpose
- **Default Experience Selection**: Defines which experience (`ULyraExperienceDefinition`) should be loaded when the map starts, if no other experience is specified externally.
- **Editor Configuration**: In PIE, can force standalone network mode for front-end levels or similar single-player scenarios.
- **Validation**: Provides error logging and checks (in editor builds) to ensure correct usage (e.g., warns if using a basic `APlayerStart` instead of a `ALyraPlayerStart`).

---

## 4. Dependencies & Relationships
- **Inherits From**: `AWorldSettings`, which holds level-specific settings in Unreal.
- **References**:
  - `ULyraExperienceDefinition`: The default experience for gameplay if no user-facing experience overrides it.
  - `APlayerStart` vs. `ALyraPlayerStart`: Editor warnings are emitted if the map uses a generic `APlayerStart` rather than the Lyra-specific variant.

The settings are typically read by `ALyraGameMode` (or similar classes) to determine which experience to load automatically.

---

## 5. Important Members

1. **`TSoftClassPtr<ULyraExperienceDefinition> DefaultGameplayExperience`**  
   - Soft reference to the default experience asset.  
   - Resolved at runtime into a valid `ULyraExperienceDefinition`.  
   - If it fails to resolve, a warning is logged.

2. **`bool ForceStandaloneNetMode`** *(Editor-only)*  
   - Forces the map to run in standalone mode when using PIE if `true`.  
   - Useful for front-end menus or levels where multiplayer logic isn’t required.

### Methods
1. **`FPrimaryAssetId GetDefaultGameplayExperience() const`**  
   - Returns a valid asset ID for the experience, if it’s specified and can be resolved.  
   - Logs an error if the path can’t be resolved to a valid asset.

2. **`void CheckForErrors() override`** *(Editor-only)*  
   - Overrides the standard map check to warn about using a normal `APlayerStart` instead of `ALyraPlayerStart`.  
   - Also intended to handle potential issues with the `DefaultGameplayExperience` reference (e.g., unscanned directories).

---

## 6. Implementation Notes & Lifecycle
- **Construction**: Created automatically when a level with this world settings is loaded.  
- **Editor-Only Checks**: `CheckForErrors()` runs in the editor to detect potential setup issues (e.g., spawning points or asset references).  
- **Experience Resolution**: The `GetDefaultGameplayExperience()` method uses `UAssetManager::Get().GetPrimaryAssetIdForPath(...)` to translate the soft class path to a `FPrimaryAssetId`. If unsuccessful, a log entry is created.

---

## 7. Example Usage

```cpp
// When launching a server for a given map, something like ALyraGameMode might do:
ALyraWorldSettings* LyraWS = Cast<ALyraWorldSettings>(World->GetWorldSettings());
if (LyraWS)
{
    FPrimaryAssetId ExperienceId = LyraWS->GetDefaultGameplayExperience();
    // If ExperienceId is valid, use it to load the default experience
}
```
In the Unreal Editor, you’d assign `ALyraWorldSettings` as the World Settings for your map, then set `DefaultGameplayExperience` in the details panel.

---

### 8. Common Pitfalls & Edge Cases

- **Unresolved `DefaultGameplayExperience`**  
  If the asset path is invalid or not included in asset scanning rules, `GetDefaultGameplayExperience()` returns an invalid ID and logs an error.

- **`APlayerStart` vs. `ALyraPlayerStart`**  
  Using the default `APlayerStart` spawns an editor warning. Switching to `ALyraPlayerStart` ensures you leverage Lyra’s extended spawn logic.

- **Forgetting to Use `ALyraWorldSettings`**  
  If the level uses standard `AWorldSettings`, the default experience logic might be skipped entirely.

---

### 9. Future Improvements or TODOs

- **Extended Editor Tools**  
  Could automatically replace `APlayerStart` with `ALyraPlayerStart` or provide a “Fix Now” button.

- **More Detailed Validation**  
  Checking other aspects like lighting settings, collision channels, or specific gameplay assets could be beneficial for large-scale projects.

---

### 10. FAQs / Troubleshooting

**Q**: Do I have to set `ALyraWorldSettings` for every level?  
**A**: If you want to define a unique default experience for each level, yes. Otherwise, a global fallback is used if no level-specific experience is found.

**Q**: I keep getting a warning about “unable to resolve an asset ID” — what does that mean?  
**A**: Ensure the path in `DefaultGameplayExperience` points to a valid `ULyraExperienceDefinition`. Check that the asset is part of an asset directory scanned by the `UAssetManager`.

**Q**: What happens if multiple experiences are specified for a single map?  
**A**: Typically, only the `DefaultGameplayExperience` is used unless overridden by a user-facing experience. The engine doesn’t natively merge them.

**Q**: Can I use `ALyraWorldSettings` for a front-end map?  
**A**: Yes, and if `ForceStandaloneNetMode` is true, you can ensure that PIE runs in standalone mode for a non-game (e.g., main menu) level.



# ULyraAssetManager

## 1. Class/Struct Name
**ULyraAssetManager**  
> A custom `UAssetManager` subclass for loading, managing, and tracking game-specific assets in Lyra. This singleton helps coordinate startup tasks, gameplay data loading, and synchronous asset fetching.

---

## 2. Overview
`ULyraAssetManager` overrides `UAssetManager` to provide Lyra-specific features such as:
- Managing global game data (e.g., default pawn, game definitions).
- Tracking “startup jobs” that run when the game boots (e.g., loading global data assets).
- Offering synchronous loading utilities (`GetAsset` and `GetSubclass`) for code that needs immediate access to data or classes.
- Keeping references to loaded assets to prevent them from being garbage-collected.

This class is critical for ensuring the game’s core data is loaded before gameplay begins and for providing utility methods to load assets on demand.

---

## 3. Key Responsibilities / Purpose
1. **Global Game Data Loading**: Provides methods to retrieve or load critical assets like `ULyraGameData` and `ULyraPawnData`.  
2. **Startup Jobs**: Maintains and executes a list of tasks (`FLyraAssetManagerStartupJob`) to initialize the game (e.g., setting up gameplay cue managers).  
3. **Synchronous Asset Access**: Supplies templated methods (`GetAsset`, `GetSubclass`) to force-load assets or blueprint classes if they’re not already loaded.  
4. **In-Editor and PIE Support**: Handles special loading logic during PIE sessions, ensuring all required data is ready for testing.

---

## 4. Dependencies & Relationships
- **Inherits From**: `UAssetManager`.  
- **Interacts With**:  
  - `ULyraGameplayCueManager`: Loads always-loaded gameplay cues.  
  - `ULyraGameData`, `ULyraPawnData`: Holds references to these essential game data assets.  
  - `UCommonSessionSubsystem`, `ULyraReplaySubsystem`: Indirectly via the data it loads, though not directly in this class.  
  - `FLyraAssetManagerStartupJob`: Used to define startup tasks with progress tracking.

`ULyraAssetManager` is globally accessible through `GEngine->AssetManager` once the engine is initialized.

---

## 5. Important Members

### Variables
1. **`LyraGameDataPath`** *(TSoftObjectPtr\<ULyraGameData\>)*  
   - Points to the global game data asset to be loaded at startup.

2. **`DefaultPawnData`** *(TSoftObjectPtr\<ULyraPawnData\>)*  
   - The default pawn data to load if none is specified on the player state.

3. **`LoadedAssets`** *(TSet\<TObjectPtr\<const UObject\>\>)*  
   - A set of assets that have been force-loaded and tracked to prevent garbage collection.

4. **`StartupJobs`** *(TArray\<FLyraAssetManagerStartupJob\>)*  
   - List of tasks to run during `StartInitialLoading`.  
   - Each job can have a weight and a callback for partial progress updates.

### Methods
1. **`static ULyraAssetManager& Get()`**  
   - Singleton accessor. Returns the global `ULyraAssetManager`. Crashes if not properly set.

2. **`StartInitialLoading()`** *(override)*  
   - Called by engine at startup. Gathers and executes all startup jobs (loading game data, initializing cues, etc.).  

3. **`SynchronousLoadAsset(const FSoftObjectPath&)`** *(static)*  
   - Internal utility that uses the Streamable Manager (or `LoadObject`) to load an asset immediately.  

4. **`AddLoadedAsset(const UObject* Asset)`**  
   - Thread-safe method to add an asset to the `LoadedAssets` set, preventing GC.

5. **`GetGameData()`**, **`GetDefaultPawnData()`**  
   - Returns references to critical game data. Loads them synchronously if necessary.

6. **Templated methods:**  
   - **`GetAsset<T>(...)`**  
   - **`GetSubclass<T>(...)`**  
   - Provide synchronous retrieval of assets or class references.

7. **`DumpLoadedAssets()`** *(static)*  
   - Debug console command that logs all assets in `LoadedAssets`.

8. **`DoAllStartupJobs()`**  
   - Executes each `FLyraAssetManagerStartupJob` in sequence, optionally providing progress updates.

---

## 6. Implementation Notes & Lifecycle
- **Initialization**  
  - The engine sets `GEngine->AssetManager` to this class based on `AssetManagerClassName` in `DefaultEngine.ini`.  
  - `StartInitialLoading` is then called automatically, running any queued startup jobs (e.g., loading game data assets).
- **Synchronous Loading**  
  - Typically discouraged in production for large assets, but used here for guaranteed immediate access.  
  - For performance, only recommended for small or critical assets (like blueprint classes).
- **Loaded Asset Tracking**  
  - `LoadedAssets` set ensures that forcibly loaded assets remain in memory. Make sure to remove or handle them carefully if memory usage is a concern.
- **Editor Flow**  
  - In the editor, `PreBeginPIE` is called to perform further loading tasks, ensuring test sessions have the correct data.

---

## 7. Example Usage

```cpp
// Example: Retrieve default pawn data in game code
void SomeFunction()
{
    const ULyraPawnData* PawnData = ULyraAssetManager::Get().GetDefaultPawnData();
    if (PawnData)
    {
        // Use PawnData for something
    }
}

// Example: Synchronously load a custom asset
TSoftObjectPtr<UStaticMesh> MyMeshPtr = ...; // from some data
UStaticMesh* MyMesh = ULyraAssetManager::GetAsset(MyMeshPtr, true);
if (MyMesh)
{
    // MyMesh is now loaded and won't be garbage collected
}
```
In practice, developers call these methods anytime they need guaranteed availability of certain data or want to define new startup tasks.

---

### 8. Common Pitfalls & Edge Cases

- **Excessive Synchronous Loads**  
  Could slow down load times if too many large assets are loaded at once.

- **Failing to Configure `AssetManagerClassName`**  
  If `ULyraAssetManager` isn’t set in `DefaultEngine.ini`, the engine can’t instantiate it, causing a fatal error.

- **Unscanned Assets**  
  If a path isn’t known to the asset manager, calls to load or retrieve the asset might fail or return null.

- **Memory Usage**  
  Adding too many assets to `LoadedAssets` could inflate memory usage unnecessarily.

---

### 9. Future Improvements or TODOs

- **Refine Startup Job System**  
  Could include priority or concurrency for certain tasks, or a more robust progress UI.

- **Asynchronous Alternative**  
  Offer more asynchronous approaches to loading assets so the main thread isn’t blocked.

- **Automated Unloading**  
  A system to remove references from `LoadedAssets` after certain usage thresholds, preventing indefinite memory usage.

---

### 10. FAQs / Troubleshooting

**Q**: Why does my game crash at startup saying it can’t find `ULyraAssetManager`?  
**A**: Check that `[/Script/Engine.Engine] AssetManagerClassName=/Script/LyraGame.LyraAssetManager` is set in your `DefaultEngine.ini`.

**Q**: When should I use `GetAsset` vs. normal asynchronous loading?  
**A**: Use `GetAsset` for small assets or those required immediately at runtime. For bigger content, you might prefer an async approach to avoid blocking.

**Q**: Does the manager automatically unload assets at any point?  
**A**: Not by default. Assets added to `LoadedAssets` remain in memory until the engine closes or you remove them manually.

**Q**: Can I add my own startup jobs?  
**A**: Yes, you could modify `ULyraAssetManager` to push additional `FLyraAssetManagerStartupJob` entries before `DoAllStartupJobs()` is called.



# FLyraAssetManagerStartupJob

## 1. Class/Struct Name
**FLyraAssetManagerStartupJob**  
> A lightweight struct that represents a single “startup job” for Lyra’s asset management system, encapsulating a task function, weight (importance), and progress reporting.

---

## 2. Overview
`FLyraAssetManagerStartupJob` holds metadata and logic for a startup task. These tasks are typically queued by the `ULyraAssetManager` to execute during the game’s initial loading phase. Each job can optionally spawn a `FStreamableHandle` to load assets asynchronously. The struct also provides hooks for reporting partial progress back to any listening system, making it useful for incrementally updating loading screens or logs.

---

## 3. Key Responsibilities / Purpose
- **Task Definition**: Bundles a function (`JobFunc`) that performs some asset-loading or initialization work.
- **Progress Tracking**: Allows partial progress to be reported via `SubstepProgressDelegate`.
- **Streamable Handle Management**: If the job spawns a `FStreamableHandle`, it can observe and wait for that handle’s completion.

---

## 4. Dependencies & Relationships
- **Used By**: `ULyraAssetManager` in its `StartupJobs` array.
- **Relies On**: 
  - `FStreamableHandle`: Unreal’s handle for asynchronous asset loads.
  - `FPlatformTime` for time measurements.
  - `FLyraAssetManagerStartupJobSubstepProgress` delegate to report intermediate progress.

---

## 5. Important Members

1. **`TFunction<void(const FLyraAssetManagerStartupJob&, TSharedPtr<FStreamableHandle>&)> JobFunc`**  
   - The primary function that performs the job’s work.  
   - Can optionally produce a `FStreamableHandle` for asynchronous loads.

2. **`FString JobName`**  
   - A descriptive name used in logs.

3. **`float JobWeight`**  
   - Relative weight or importance for progress tracking (e.g., a job might be heavier if it loads more assets).

4. **`FLyraAssetManagerStartupJobSubstepProgress SubstepProgressDelegate`**  
   - Delegate used to update the job’s substep progress.  
   - Bound externally to, for example, `ULyraAssetManager` for incremental loading progress.

5. **`mutable double LastUpdate`**  
   - Tracks the last timestamp when substep progress was reported, reducing overhead by limiting frequent calls.

---

## 6. Implementation Notes & Lifecycle
- **Job Execution**  
  - The `DoJob()` method logs the start time, calls `JobFunc`, and, if a handle is created, waits for it to complete.  
  - The `FStreamableHandle::BindUpdateDelegate` is temporarily set to report incremental progress, then cleared once the handle finishes.
- **Progress Updates**  
  - `UpdateSubstepProgress(float NewProgress)` calls the bound delegate directly.  
  - `UpdateSubstepProgressFromStreamable` checks if enough time (`> 1/60th of a second`) has passed to avoid excessive calls.  
- **Integration**  
  - Typically invoked by `ULyraAssetManager::DoAllStartupJobs()`, which runs these in sequence or accumulates progress.

---

## 7. Example Usage

```cpp
// Creating a startup job in ULyraAssetManager constructor or StartInitialLoading
StartupJobs.Add(FLyraAssetManagerStartupJob(
    TEXT("LoadSomeAssets"),
    [this](const FLyraAssetManagerStartupJob& Job, TSharedPtr<FStreamableHandle>& Handle)
    {
        // Example: Use the StreamableManager to load assets asynchronously
        //   (populate Handle with LoadAsset call if needed)
    },
    10.0f // Weighted more heavily
));
```
During the job’s execution, the system might bind a substep delegate that reports progress to a loading screen or debug log.

---

### 8. Common Pitfalls & Edge Cases

- **Never Creating a `FStreamableHandle`**  
  If `JobFunc` doesn’t create a handle, `DoJob()` will simply run synchronously. That’s fine if the job doesn’t actually load assets.

- **Excessive Update Frequency**  
  The `LastUpdate` check prevents flooding calls to `UpdateSubstepProgressFromStreamable`. Otherwise, it could degrade performance.

- **Inconsistent Weight**  
  If `JobWeight` is too large or too small relative to others, the UI might display misleading load percentages.

---

### 9. Future Improvements or TODOs

- **Parallel Job Execution**  
  Currently, `DoJob()` is synchronous. Might explore concurrency or partial overlap of job tasks.

- **Better Time-Based Throttling**  
  Provide a user-configurable threshold for how frequently to update the substep progress.

- **Error Handling**  
  If an asset fails to load, `JobFunc` might want to handle or log that in a user-friendly way.

---

### 10. FAQs / Troubleshooting

**Q**: Do I have to always return a `FStreamableHandle` from `JobFunc`?  
**A**: No, if your job doesn’t involve asynchronous asset loading, it can run synchronously and set `Handle` to `nullptr`.

**Q**: How often can substep progress be updated?  
**A**: Default logic only updates once every ~1/60th of a second. You can adjust or remove this check if needed.

**Q**: What if I need multiple tasks in a single job?  
**A**: You can implement that logic within `JobFunc`, potentially chaining multiple handle loads or tasks before returning.

**Q**: Is the `JobWeight` mandatory?  
**A**: The system uses it to calculate relative progress if there are multiple jobs. You can set it to `1.0f` or any value that makes sense in your progress scale.
