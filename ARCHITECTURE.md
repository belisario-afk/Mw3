# KillaDome.cs - Architecture Diagram

## System Architecture

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         KILLADOME PLUGIN (Main Class)                    │
│                              Version 1.0.0                               │
└────────────────┬────────────────────────────────────────────────────────┘
                 │
    ┌────────────┴────────────┐
    │                         │
    ▼                         ▼
┌───────────────────┐   ┌─────────────────────────┐
│  CONFIGURATION    │   │   PLAYER SESSIONS       │
│     LAYER         │   │      (Runtime)          │
├───────────────────┤   ├─────────────────────────┤
│ PluginConfig      │   │ Dictionary<ulong,       │
│ GunConfig         │   │   PlayerSession>        │
│ OutfitConfig      │   │                         │
│                   │   │ - Player reference      │
│ - Spawn points    │   │ - Profile data          │
│ - Economy values  │   │ - UI state              │
│ - Progression     │   │ - Pagination            │
│ - Performance     │   │                         │
└───────────────────┘   └─────────────────────────┘
         │                        │
         └────────┬───────────────┘
                  │
    ┌─────────────┴─────────────┐
    │                           │
    ▼                           ▼
┌────────────────────────┐  ┌──────────────────────────┐
│   CORE SYSTEMS         │  │   SECURITY LAYER         │
│   (11 Modules)         │  │                          │
└────────────────────────┘  └──────────────────────────┘
```

## Module Dependencies

```
                     ┌─────────────────┐
                     │   KillaDome     │
                     │   (Main Class)  │
                     └────────┬────────┘
                              │
                              │ creates & manages
                              │
         ┌────────────────────┼────────────────────┐
         │                    │                    │
         ▼                    ▼                    ▼
   ┌──────────┐        ┌──────────┐        ┌──────────┐
   │  Lobby   │        │  Dome    │        │  Save    │
   │   UI     │        │ Manager  │        │ Manager  │
   └────┬─────┘        └─────┬────┘        └────┬─────┘
        │                    │                   │
        │uses                │uses               │saves/loads
        │                    │                   │
   ┌────▼──────────────┐    │             ┌─────▼──────┐
   │  LoadoutEditor    │    │             │  Player    │
   │  StoreAPI         │    │             │  Profile   │
   │  ForgeStation     │    │             └────────────┘
   └───────────────────┘    │
                            │
        ┌───────────────────┴──────────────────┐
        │                                       │
        ▼                                       ▼
   ┌──────────────┐                      ┌────────────┐
   │  Weapon      │◄─────────────────────┤  Token     │
   │  Progression │                      │  Economy   │
   └──────┬───────┘                      └─────┬──────┘
          │                                    │
          │uses                                │uses
          │                                    │
   ┌──────▼───────┐                      ┌────▼──────┐
   │  Attachment  │                      │  Store    │
   │   System     │                      │   API     │
   └──────────────┘                      └───────────┘
          │
          │applies to
          │
   ┌──────▼───────┐
   │   VFX/SFX    │
   │   Managers   │
   └──────────────┘
```

## Data Flow: Player Connection

```
Player Connects
     │
     ▼
OnPlayerConnected Hook
     │
     ├──► Load Profile from JSON (SaveManager)
     │         │
     │         ├─► File exists? → Deserialize JSON
     │         └─► No file? → Create new profile
     │
     ├──► Create PlayerSession in memory
     │         │
     │         └─► Store in _activeSessions dictionary
     │
     ├──► Teleport to Lobby
     │         │
     │         └─► player.Teleport(LobbySpawnPosition)
     │
     └──► Show Lobby UI (1 second delay)
               │
               └─► Build CUI elements → Send to client
```

## Data Flow: Loadout System

```
┌────────────────────────────────────────────────────────┐
│                  LOADOUT FLOW                          │
└────────────────────────────────────────────────────────┘

1. EDIT LOADOUT (In Lobby)
   │
   ├─► Player opens Loadouts Tab
   │      │
   │      └─► Display current loadout from PlayerProfile
   │
   ├─► Player clicks "Next Weapon"
   │      │
   │      └─► killadome.weapon.next command
   │             │
   │             ├─► CycleWeapon() modifies loadout
   │             │      │
   │             │      └─► Increment weapon index
   │             │
   │             └─► SaveManager.SavePlayerProfile()
   │
   ├─► Player purchases skin
   │      │
   │      └─► killadome.purchase command
   │             │
   │             ├─► Check tokens (BloodTokenEconomy)
   │             ├─► Deduct tokens
   │             ├─► Add to OwnedSkins list
   │             └─► Save profile
   │
   └─► Player applies skin
          │
          └─► killadome.applyskin command
                 │
                 ├─► Check ownership
                 ├─► Update Loadout.Skins dictionary
                 └─► Save profile

2. APPLY LOADOUT (Enter Arena)
   │
   ├─► Player clicks "Join Queue"
   │      │
   │      └─► DomeManager.AddToQueue()
   │
   ├─► Match starts
   │      │
   │      └─► TeleportToArena()
   │             │
   │             └─► ApplyLoadout() ◄──── CRITICAL POINT
   │
   └─► ApplyLoadout Process:
          │
          ├─► Strip all items
          │      └─► player.inventory.Strip()
          │
          ├─► Give Primary Weapon
          │      ├─► Create item by shortname
          │      ├─► Apply skin (if owned)
          │      ├─► Add attachments to contents
          │      └─► Give ammo
          │
          ├─► Give Secondary Weapon
          │      └─► (same as primary)
          │
          └─► Give Armor
                 ├─► Head: metal.facemask
                 ├─► Chest: metal.plate.torso
                 ├─► Legs: heavy.plate.pants
                 ├─► Hands: tactical.gloves
                 └─► Feet: shoes.boots
```

## Data Flow: Store Purchase

```
┌────────────────────────────────────────────────────────┐
│              STORE PURCHASE FLOW                       │
└────────────────────────────────────────────────────────┘

Player clicks BUY button
     │
     ▼
killadome.purchase <itemId> <cost>
     │
     ├──► Rate Limit Check (AntiExploit)
     │       │
     │       └─► Allow max 5 actions/second
     │
     ├──► Validate Session
     │       │
     │       └─► GetSession() or create new
     │
     ├──► Check Balance
     │       │
     │       ├─► profile.Tokens >= cost?
     │       │
     │       ├─── NO ──► SendReply("Insufficient tokens")
     │       │
     │       └─── YES ──► Continue
     │
     ├──► Process Purchase (StoreAPI)
     │       │
     │       ├─► BloodTokenEconomy.SpendTokens()
     │       │      │
     │       │      ├─► Deduct tokens
     │       │      └─► Return success/fail
     │       │
     │       └─── SUCCESS ──► Add to OwnedSkins
     │
     ├──► Save Profile (SaveManager)
     │       │
     │       ├─► Serialize to JSON
     │       ├─► Write to .tmp file
     │       ├─► Atomic swap to real file
     │       └─► Delete .tmp
     │
     ├──► Refresh UI
     │       │
     │       └─► ShowLobbyUIWithTab(player, "store")
     │
     └──► SendReply("Successfully purchased!")
```

## UI Architecture

```
┌───────────────────────────────────────────────────────────────┐
│                     RUST CLIENT SCREEN                        │
│  ┌─────────────────────────────────────────────────────────┐  │
│  │               UI_MAIN (Root Container)                   │  │
│  │  ┌───────────────────────────────────────────────────┐   │  │
│  │  │         UI_HEADER (Player Info Bar)               │   │  │
│  │  │  [Player Name]        [◆ 1,250 Tokens]  [✖ Close] │   │  │
│  │  └───────────────────────────────────────────────────┘   │  │
│  │                                                           │  │
│  │  ┌───────────────────────────────────────────────────┐   │  │
│  │  │         UI_TABS (Navigation Bar)                  │   │  │
│  │  │  [PLAY] [LOADOUTS] [STORE] [STATS] [SETTINGS]    │   │  │
│  │  └───────────────────────────────────────────────────┘   │  │
│  │                                                           │  │
│  │  ┌───────────────────────────────────────────────────┐   │  │
│  │  │         UI_TAB_CONTAINER (Content Area)           │   │  │
│  │  │                                                   │   │  │
│  │  │  ┌─────────────────────────────────────────────┐ │   │  │
│  │  │  │  DYNAMIC CONTENT BASED ON SELECTED TAB      │ │   │  │
│  │  │  │                                             │ │   │  │
│  │  │  │  • Play Tab → Matchmaking                   │ │   │  │
│  │  │  │  • Loadouts Tab → Weapon Customizer         │ │   │  │
│  │  │  │  • Store Tab → Shop (3 sub-categories)      │ │   │  │
│  │  │  │  • Stats Tab → Player Statistics            │ │   │  │
│  │  │  │  • Settings Tab → Plugin Info               │ │   │  │
│  │  │  │                                             │ │   │  │
│  │  │  └─────────────────────────────────────────────┘ │   │  │
│  │  │                                                   │   │  │
│  │  └───────────────────────────────────────────────────┘   │  │
│  └─────────────────────────────────────────────────────────┘  │
└───────────────────────────────────────────────────────────────┘
```

## Store Tab Structure

```
┌─────────────────────────────────────────────────────────────┐
│                      STORE TAB                              │
├─────────────────────────────────────────────────────────────┤
│  [🩸 Blood Tokens: 1,250]                  [Balance Display]│
├─────────────────────────────────────────────────────────────┤
│  [⚔ GUN STORE] [🎨 SKINS STORE] [👕 OUTFIT STORE]  ← Sub-tabs
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  When "GUN STORE" selected:                                │
│  ┌───────────────────┬───────────────────────────────────┐ │
│  │  BASE GUNS        │  ATTACHMENTS                      │ │
│  │  (Left 45%)       │  (Right 45%)                      │ │
│  │                   │                                   │ │
│  │  • AK-47          │  ┌─────┬─────┬─────┐             │ │
│  │  • LR-300         │  │Scope│Scope│Scope│  ← 3 cols   │ │
│  │  • M249           │  ├─────┼─────┼─────┤             │ │
│  │  • MP5            │  │Laser│Silnc│Silnc│             │ │
│  │  • Thompson       │  ├─────┼─────┼─────┤             │ │
│  │  • Python         │  │Silnc│Brake│Boost│             │ │
│  │  [◀ PREV] [NEXT ▶]│  └─────┴─────┴─────┘             │ │
│  └───────────────────┴───────────────────────────────────┘ │
│                                                             │
│  When "SKINS STORE" selected:                              │
│  ┌─────────────────────────────────────────────────────┐   │
│  │         WEAPON & OUTFIT SKINS (12 per page)         │   │
│  │                                                     │   │
│  │  ┌──────┬──────┬──────┬──────┐                     │   │
│  │  │ Skin │ Skin │ Skin │ Skin │  ← Row 1 (4 items)  │   │
│  │  ├──────┼──────┼──────┼──────┤                     │   │
│  │  │ Skin │ Skin │ Skin │ Skin │  ← Row 2            │   │
│  │  ├──────┼──────┼──────┼──────┤                     │   │
│  │  │ Skin │ Skin │ Skin │ Skin │  ← Row 3            │   │
│  │  └──────┴──────┴──────┴──────┘                     │   │
│  │                                                     │   │
│  │  [◀ PREV]                      [NEXT ▶]   Page 1/3 │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
│  When "OUTFIT STORE" selected:                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │           ARMOR PIECES (12 per page)                │   │
│  │  ┌──────┬──────┬──────┬──────┐                     │   │
│  │  │ Head │ Head │Chest │Chest │                     │   │
│  │  ├──────┼──────┼──────┼──────┤                     │   │
│  │  │ Legs │ Legs │Hands │ Feet │                     │   │
│  │  └──────┴──────┴──────┴──────┘                     │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
├─────────────────────────────────────────────────────────────┤
│  💎 Purchase items to enhance your loadout                 │
└─────────────────────────────────────────────────────────────┘
```

## Loadouts Tab Structure

```
┌─────────────────────────────────────────────────────────────┐
│                    LOADOUTS TAB                             │
├─────────────────────────────────────────────────────────────┤
│  [⚔ WEAPONS] [👕 OUTFIT]                      ← Sub-tabs   │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  When "WEAPONS" selected:                                  │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  PRIMARY WEAPON                                     │   │
│  │  ┌───────────────────────────────────────────────┐  │   │
│  │  │  [◀]  [AK-47 Image]  [▶]                      │  │   │
│  │  │       AK-47                                    │  │   │
│  │  │  [EDIT] ← Opens attachment selector           │  │   │
│  │  └───────────────────────────────────────────────┘  │   │
│  │                                                     │   │
│  │  SECONDARY WEAPON                                  │   │
│  │  ┌───────────────────────────────────────────────┐  │   │
│  │  │  [◀]  [Python Image]  [▶]                     │  │   │
│  │  │       Python Revolver                         │  │   │
│  │  │  [EDIT]                                       │  │   │
│  │  └───────────────────────────────────────────────┘  │   │
│  │                                                     │   │
│  │  When EDIT clicked:                                │   │
│  │  ┌───────────────────────────────────────────────┐  │   │
│  │  │  ATTACHMENTS                                  │  │   │
│  │  │  [Scopes] [Silencers] [Underbarrel]          │  │   │
│  │  │                                               │  │   │
│  │  │  • 8x Scope        [EQUIP]                   │  │   │
│  │  │  • Holo Sight      [EQUIP]                   │  │   │
│  │  │  • Red Dot         [EQUIP]                   │  │   │
│  │  └───────────────────────────────────────────────┘  │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
│  When "OUTFIT" selected:                                   │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  HEAD:   [◀] [Metal Facemask] [▶]                  │   │
│  │  CHEST:  [◀] [Metal Plate]    [▶]                  │   │
│  │  LEGS:   [◀] [Heavy Pants]    [▶]                  │   │
│  │  HANDS:  [◀] [Tact. Gloves]   [▶]                  │   │
│  │  FEET:   [◀] [Heavy Boots]    [▶]                  │   │
│  └─────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

## Database Schema (JSON Files)

```
PlayerProfile.json Structure:
┌─────────────────────────────────────────────┐
│ PlayerProfile                               │
├─────────────────────────────────────────────┤
│ - SteamID: ulong                            │
│ - Tokens: int ───────┐                      │
│ - TotalKills: int    │  Stats              │
│ - TotalDeaths: int   │                      │
│ - MatchesPlayed: int │                      │
│ - IsVIP: bool ───────┘                      │
│                                             │
│ - Loadouts: List<Loadout> ──┐              │
│   └── Loadout                │  Equipment  │
│       - Name                 │             │
│       - Primary              │             │
│       - Secondary            │             │
│       - Attachments          │             │
│       - Skins                │             │
│       - Armor pieces ────────┘             │
│                                             │
│ - OwnedSkins: List<string> ──┐             │
│ - OwnedArmor: List<string>   │  Unlocks   │
│ - WeaponLevels: Dict         │             │
│ - AttachmentLevels: Dict ────┘             │
│                                             │
│ - LastUpdated: DateTime                    │
└─────────────────────────────────────────────┘
```

## State Management

```
┌────────────────────────────────────────────────┐
│           PLUGIN STATE LAYERS                  │
├────────────────────────────────────────────────┤
│                                                │
│  LAYER 1: Configuration (Static)              │
│  ┌──────────────────────────────────────────┐ │
│  │ PluginConfig (from JSON)                 │ │
│  │ GunConfig (code-defined)                 │ │
│  │ OutfitConfig (code-defined)              │ │
│  └──────────────────────────────────────────┘ │
│  ↓ Loaded once on Init()                      │
│                                                │
│  LAYER 2: Runtime Sessions (Memory)           │
│  ┌──────────────────────────────────────────┐ │
│  │ _activeSessions: Dictionary              │ │
│  │   Key: SteamID (ulong)                   │ │
│  │   Value: PlayerSession                   │ │
│  │     - Player reference                   │ │
│  │     - Profile (persistent data)          │ │
│  │     - UI state (temporary)               │ │
│  └──────────────────────────────────────────┘ │
│  ↓ Created on connect, destroyed on disconnect│
│                                                │
│  LAYER 3: Persistent Storage (Disk)           │
│  ┌──────────────────────────────────────────┐ │
│  │ JSON files (one per player)              │ │
│  │   /oxide/data/KillaDome/{steamid}.json   │ │
│  │                                           │ │
│  │ Saved:                                    │ │
│  │   - On disconnect                         │ │
│  │   - Every 5 minutes (auto-save)          │ │
│  │   - After purchases/changes              │ │
│  └──────────────────────────────────────────┘ │
│                                                │
└────────────────────────────────────────────────┘
```

## Security Architecture

```
┌──────────────────────────────────────────────────┐
│             SECURITY LAYERS                      │
├──────────────────────────────────────────────────┤
│                                                  │
│  LAYER 1: Rate Limiting (AntiExploit)           │
│  ┌────────────────────────────────────────────┐ │
│  │ RateLimiter per player                     │ │
│  │   - Max 5 actions per second (default)     │ │
│  │   - Rolling window (Queue<DateTime>)       │ │
│  │   - Blocks: purchases, weapon changes      │ │
│  └────────────────────────────────────────────┘ │
│  ↓ Prevents spam/exploitation                   │
│                                                  │
│  LAYER 2: Input Validation                      │
│  ┌────────────────────────────────────────────┐ │
│  │ • Balance checks before spending           │ │
│  │ • Ownership checks before equipping        │ │
│  │ • Level caps on upgrades                   │ │
│  │ • Null reference checks                    │ │
│  └────────────────────────────────────────────┘ │
│  ↓ Prevents invalid operations                  │
│                                                  │
│  LAYER 3: Permission System                     │
│  ┌────────────────────────────────────────────┐ │
│  │ Oxide permissions:                         │ │
│  │   - killadome.admin (admin commands)       │ │
│  │   - killadome.vip (VIP features)           │ │
│  └────────────────────────────────────────────┘ │
│  ↓ Restricts privileged operations              │
│                                                  │
│  LAYER 4: Data Integrity                        │
│  ┌────────────────────────────────────────────┐ │
│  │ • Atomic file writes (.tmp swap)           │ │
│  │ • Exception handling with fallbacks        │ │
│  │ • Corruption detection (try/catch)         │ │
│  └────────────────────────────────────────────┘ │
│  ↓ Ensures data consistency                     │
│                                                  │
└──────────────────────────────────────────────────┘
```

## Performance Architecture

```
┌──────────────────────────────────────────────────┐
│          PERFORMANCE OPTIMIZATIONS               │
├──────────────────────────────────────────────────┤
│                                                  │
│  1. UI Update Throttling                        │
│     • Configurable delay (100ms default)        │
│     • Prevents rapid rebuilds                   │
│                                                  │
│  2. Pagination                                   │
│     • Skins: 12 per page                        │
│     • Guns: 6 per page                          │
│     • Reduces element count                     │
│                                                  │
│  3. Lazy Loading                                 │
│     • Images loaded after 5s delay              │
│     • Waits for ImageLibrary ready              │
│                                                  │
│  4. Efficient Data Structures                    │
│     • Dictionary<ulong, Session> for O(1) lookup│
│     • List.Skip().Take() for pagination         │
│                                                  │
│  5. Minimal Garbage Collection                   │
│     • StringBuilder for string concat (TODO)    │
│     • Object pooling opportunities (TODO)       │
│     • Reuse CuiElementContainer (TODO)          │
│                                                  │
│  6. Async Operations                             │
│     • File I/O in background (atomic writes)    │
│     • Timer-based auto-save                     │
│                                                  │
└──────────────────────────────────────────────────┘
```

---

**Document Version:** 1.0  
**Last Updated:** 2024-11-23  
**Plugin Version:** 1.0.0

For detailed code analysis, see: `KILLADOME_ANALYSIS.md`  
For quick tasks, see: `QUICK_REFERENCE.md`
