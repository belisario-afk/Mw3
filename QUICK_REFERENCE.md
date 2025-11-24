# KillaDome.cs - Quick Reference Guide

## 🎯 Quick Navigation

| Need to... | Go to Line(s) | Section |
|------------|---------------|---------|
| Add a new weapon | 84-156 | GunConfig.Guns |
| Add a new skin | 168-261 | GunConfig.Skins |
| Add armor piece | 309-400 | OutfitConfig.Armors |
| Change economy values | Config file | StartingTokens, TokensPerKill |
| Modify UI layout | 2150-3588 | LobbyUI class |
| Add attachment | 3662-3701 | AttachmentSystem |
| Fix save/load issues | 4027-4098 | SaveManager |
| Adjust security | 4102-4166 | AntiExploit |
| Add console command | 880-1390 | Console Commands region |

---

## 🚀 Common Tasks

### Adding a New Weapon (2 minutes)

**File:** KillaDome.cs  
**Line:** ~84

```csharp
// ADD THIS to the Guns dictionary:
["mygun"] = new GunDefinition
{
    Id = "mygun",
    DisplayName = "My Awesome Gun",
    RustItemShortname = "rifle.ak",  // Use any valid Rust item
    ImageUrl = "https://i.imgur.com/YOUR_IMAGE.png"
}
```

✅ **That's it!** The weapon now appears in:
- Loadout selector
- Weapon cycling
- Store (if you add skins)

### Adding a Weapon Skin (2 minutes)

**File:** KillaDome.cs  
**Line:** ~168

```csharp
// ADD THIS to the Skins list:
new SkinDefinition
{
    Name = "My Gun - Epic Skin",
    SkinId = "123456789",           // Workshop ID or custom ID
    WeaponId = "mygun",             // Must match a gun Id
    ImageUrl = "https://i.imgur.com/SKIN_IMAGE.png",
    Cost = 500,                     // Blood Tokens
    Tag = "NEW",                    // "NEW", "POPULAR", "HOT", or ""
    Rarity = "Epic"                 // "Common", "Rare", "Epic", "Legendary"
}
```

✅ Automatically appears in Skins Store tab!

### Adding Armor (2 minutes)

**File:** KillaDome.cs  
**Line:** ~309

```csharp
// ADD THIS to the Armors list:
new ArmorItem
{
    Name = "Cool Helmet",
    ItemShortname = "metal.facemask",  // Rust item shortname
    Slot = "head",                     // "head", "chest", "legs", "hands", "feet"
    SkinId = "0",
    ImageUrl = "https://i.imgur.com/ARMOR_IMAGE.png",
    Cost = 300,
    Rarity = "Rare"
}
```

✅ Automatically appears in Outfit Store!

### Changing Token Rewards (30 seconds)

**File:** oxide/config/KillaDome.json

```json
{
  "Starting Blood Tokens": 1000,    // ← Change starting balance
  "Tokens Per Kill": 25             // ← Change kill reward
}
```

✅ Reload config: `oxide.reload KillaDome`

### Changing Item Prices (1 minute)

**In Code (for skins):**
```csharp
Cost = 750,  // ← Change this number (line ~177)
```

**In Code (for armor):**
```csharp
Cost = 450,  // ← Change this number (line ~319)
```

✅ Reload plugin: `oxide.reload KillaDome`

---

## 🎨 UI Customization

### Change Colors

**Main Background:**
```csharp
Color = "0.06 0.06 0.08 0.9"  // Dark gray - Line ~2155
```

**Button Active:**
```csharp
Color = "0.2 0.6 0.8 0.9"  // Blue - Line ~2195
```

**Currency Gold:**
```csharp
Color = "1 0.8 0 1"  // Gold - Line ~2245
```

**Format:** "R G B A" (0.0 to 1.0 range)

### Change Font Sizes

```csharp
FontSize = 14  // Tab buttons
FontSize = 24  // Headers
FontSize = 10  // Small text
```

### Move UI Elements

```csharp
// Anchor format: "X Y" (0 = left/bottom, 1 = right/top)
AnchorMin = "0.3 0.4"  // Bottom-left corner
AnchorMax = "0.7 0.6"  // Top-right corner

// Example: Full screen
AnchorMin = "0 0"
AnchorMax = "1 1"

// Example: Center quarter
AnchorMin = "0.375 0.375"
AnchorMax = "0.625 0.625"
```

---

## 🔧 Configuration

### Config File Location
```
oxide/config/KillaDome.json
```

### Key Settings

```json
{
  "Lobby Spawn Position": {
    "x": 0.0,
    "y": 100.0,
    "z": 0.0
  },
  "Arena Spawn Positions": [
    {"x": 100.0, "y": 50.0, "z": 200.0},
    {"x": -100.0, "y": 50.0, "z": -200.0}
  ],
  "Starting Blood Tokens": 500,
  "Tokens Per Kill": 10,
  "Max Attachment Level": 5,
  "UI Update Throttle MS": 100,
  "Auto Save Interval Seconds": 300.0,
  "Enable Debug Logging": false
}
```

### Get Coordinates for Spawns

**In-game:**
1. Stand where you want spawn point
2. Type in F1 console: `/kd` (opens UI)
3. Note your position in server console
4. OR use: `player.position` in F1

---

## 🔐 Permissions

### Grant Permissions

**Admin:**
```
oxide.grant user PlayerName killadome.admin
oxide.grant group admin killadome.admin
```

**VIP:**
```
oxide.grant user PlayerName killadome.vip
oxide.grant group vip killadome.vip
```

### Check Permissions
```
oxide.show user PlayerName
```

---

## 🎮 Commands Reference

### Chat Commands
```
/kd                 - Show help
/kd open            - Open lobby UI
/kd stats           - View your stats
/kd help            - Show help
```

### Console Commands (Admin)
```
kd.open                           - Open UI for player
kd.start                          - Start a match
kd.giveskin <steamid> <skinid>   - Grant skin to player
kd.resetprogress <steamid>       - Reset player progress
```

### UI Commands (Auto-triggered)
```
killadome.close                   - Close UI
killadome.tab <tabname>          - Switch tab
killadome.joinqueue              - Join matchmaking
killadome.weapon.next <slot>     - Next weapon
killadome.weapon.prev <slot>     - Previous weapon
killadome.purchase <id> <cost>   - Buy item
killadome.storepage <next|prev>  - Change store page
```

---

## 🐛 Troubleshooting

### Images Not Showing
**Problem:** Black boxes instead of weapon/skin images  
**Solution:**
1. Install ImageLibrary plugin
2. Restart server
3. Wait 10 seconds after plugin load
4. Check F1 console for "Loaded X images" message

### Player Data Not Saving
**Problem:** Progress resets on reconnect  
**Solution:**
1. Check `oxide/data/KillaDome/` folder exists
2. Check file permissions (read/write)
3. Enable debug: `"Enable Debug Logging": true`
4. Check console for save errors

### UI Not Closing
**Problem:** UI stuck on screen  
**Solution:**
```
killadome.close  (in F1 console)
```
Or reload plugin:
```
oxide.reload KillaDome
```

### Rate Limit Spam
**Problem:** "Please slow down!" message  
**Solution:** Edit line ~4114
```csharp
public bool CheckRateLimit(ulong steamId, int maxActionsPerSecond = 10)
// Increase from 5 to 10 or higher
```

### Loadout Not Applying
**Problem:** No weapons when entering arena  
**Solution:**
1. Check weapon shortname is valid: `rifle.ak`, `smg.mp5`, etc.
2. Enable debug logging
3. Check console: "Failed to create weapon: X"
4. Verify GunConfig has correct RustItemShortname

---

## 📊 Data Files

### Player Data Location
```
oxide/data/KillaDome/{steamid}.json
```

### Example Player Data
```json
{
  "SteamID": 76561198012345678,
  "Tokens": 1250,
  "TotalKills": 45,
  "TotalDeaths": 32,
  "MatchesPlayed": 12,
  "IsVIP": false,
  "Loadouts": [
    {
      "Name": "Main Loadout",
      "Primary": "ak47",
      "Secondary": "python",
      "PrimaryAttachments": {
        "optic": "reflex",
        "barrel": "silencer"
      },
      "SecondaryAttachments": {},
      "Skins": {
        "ak47": "3602286295"
      },
      "ArmorHead": "metal.facemask",
      "ArmorChest": "metal.plate.torso",
      "ArmorLegs": "heavy.plate.pants",
      "ArmorHands": "tactical.gloves",
      "ArmorFeet": "shoes.boots"
    }
  ],
  "OwnedSkins": [
    "3602286295",
    "3102802323"
  ],
  "OwnedArmor": [
    "metal.facemask",
    "metal.plate.torso"
  ],
  "AttachmentLevels": {
    "silencer": 2,
    "reflex": 1
  },
  "LastUpdated": "2024-11-23T12:00:00Z"
}
```

### Backup Player Data
```bash
# Linux/Mac
cp -r oxide/data/KillaDome/ oxide/data/KillaDome_backup_$(date +%Y%m%d)/

# Windows PowerShell
Copy-Item oxide/data/KillaDome -Destination oxide/data/KillaDome_backup_$(Get-Date -Format yyyyMMdd) -Recurse
```

---

## 🎯 Performance Tips

### Reduce Lag
1. Increase UI throttle: `"UI Update Throttle MS": 200`
2. Increase auto-save interval: `"Auto Save Interval Seconds": 600`
3. Reduce image quality/size
4. Limit active players in lobby

### Memory Optimization
1. Limit owned skins per player
2. Paginate large lists (already done for skins)
3. Clear old player data periodically

### Server Load
```bash
# Check active sessions
kd.debug.sessions  (console command)

# Monitor auto-saves
# Look for "Auto-saved X player profiles" every 5 minutes
```

---

## 📝 Checklist for Updates

Before deploying changes:
- [ ] Test weapon appears in loadout
- [ ] Test weapon can be equipped
- [ ] Test skin appears in store
- [ ] Test purchase deducts tokens
- [ ] Test loadout persists after reconnect
- [ ] Test images load correctly
- [ ] Test UI opens without errors
- [ ] Check F1 console for errors
- [ ] Verify config loads
- [ ] Backup player data

---

## 🆘 Getting Help

### Enable Debug Mode
```json
"Enable Debug Logging": true
```
Reload plugin: `oxide.reload KillaDome`

### Check Console
Look for:
- `[DEBUG]` messages
- `[KillaDome]` errors
- `Failed to...` warnings

### Common Error Messages

**"Configuration file is corrupt"**
→ Delete `oxide/config/KillaDome.json`, let it regenerate

**"ImageLibrary not loaded"**
→ Install ImageLibrary plugin from uMod

**"Failed to create weapon: X"**
→ Check weapon shortname is valid Rust item

**"Insufficient tokens"**
→ Player doesn't have enough Blood Tokens

**"You don't own this skin"**
→ Grant skin: `kd.giveskin <steamid> <skinid>`

---

## 🔗 Useful Links

### Rust Item Shortnames
- https://www.corrosionhour.com/rust-item-list/
- Search for weapon names to find shortnames

### Image Hosting
- https://imgur.com/ (free image hosting)
- Upload weapon/skin images, get direct link

### Workshop Skin IDs
- https://steamcommunity.com/workshop/browse/?appid=252490
- Find skin in workshop, copy ID from URL

### Oxide/uMod Documentation
- https://umod.org/documentation
- Plugin development guides

---

## 📄 File Structure

```
oxide/
├── config/
│   └── KillaDome.json          ← Configuration
├── data/
│   └── KillaDome/
│       ├── 76561198012345678.json  ← Player data
│       ├── 76561198087654321.json
│       └── ...
├── plugins/
│   └── KillaDome.cs            ← Main plugin file
└── logs/
    └── oxide_*.txt             ← Check for errors
```

---

## 🎓 Learning Resources

### Understanding the Code
1. Read `KILLADOME_ANALYSIS.md` (comprehensive guide)
2. Start with small changes (add weapon)
3. Test changes on test server first
4. Use debug logging to track execution

### C# Basics
- Variables: `string name = "value";`
- Dictionaries: `dict["key"] = value;`
- Lists: `list.Add(item);`
- Classes: Define data structures

### Oxide Basics
- Hooks: Server events (OnPlayerConnected, etc.)
- Commands: `[ChatCommand]`, `[ConsoleCommand]`
- Timers: `timer.Once()`, `timer.Every()`
- Permissions: `permission.UserHasPermission()`

---

**Last Updated:** 2024-11-23  
**Plugin Version:** 1.0.0  
**Document Version:** 1.0

For detailed information, see: `KILLADOME_ANALYSIS.md`
