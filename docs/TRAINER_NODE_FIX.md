# Trainer Node Fix Guide

## Problem
Trainer nodes were not appearing on the minimap/worldmap when using the `/db track trainer` command.

## Root Cause
The trainer tracking feature used "meta IDs" (5032 for Alchemy, 5033 for Blacksmithing, etc.) that don't correspond to actual NPC spawns in the game world. These meta IDs exist in the database but have empty coordinate arrays:

```lua
[5032] = {
  ["coords"] = {},  -- Empty!
  ["fac"] = "AH",
  ["lvl"] = "10",
}
```

Real trainer NPCs in Classic WoW have different IDs and actual spawn locations. For example, Ainethil (ID 4160) is an actual Alchemy trainer with coordinates.

## Solution
The fix involves two enhancements:

### 1. Extractor Enhancement (`toolbox/extractor.lua`)
- Added trainer NPC flag (16) to the extraction process
- Trainers are now extracted from the game database automatically
- Real trainer NPCs with coordinates are added to `db/meta.lua`

### 2. Automatic Icon Assignment (`icons.lua`)
- Automatically detects trainer profession from NPC name
- Assigns appropriate profession icons (Alchemy, Blacksmithing, etc.)
- Ensures extracted trainers display with correct icons

## How to Apply the Fix

### Prerequisites
You need a database environment with VMaNGOS or CMaNGOS data. See `toolbox/README.md` for complete setup instructions.

**Summary of setup:**
1. Install MariaDB and Lua dependencies
2. Download and import VMaNGOS or CMaNGOS database dumps
3. Import pfQuest client data
4. Optimize database with indexes

### Running the Extractor
```bash
cd toolbox
make
```

This will:
1. Query the game database for all NPCs with trainer flags
2. Extract their coordinates and faction information  
3. Generate updated `db/meta.lua` and `db/meta-tbc.lua` files
4. Include real trainer NPCs in the trainer tracking list

### After Extraction
1. Restart WoW with the updated addon
2. Use `/db track trainer` to enable trainer tracking
3. Trainer nodes will appear on minimap and worldmap
4. Hovering shows trainer name and profession
5. Icons show the appropriate profession symbol

## Technical Details

### NPC Flags
Trainers are identified by `NpcFlags & 16` (UNIT_NPC_FLAG_TRAINER) in the database.

**Vanilla (VMaNGOS):** Flag value is 16
**TBC (CMaNGOS):** Flag value is 80

### Icon Matching
Icons are assigned based on keywords in trainer names:
- "Alch" → Alchemy icon
- "Blacksmith" → Blacksmithing icon  
- "Cook" → Cooking icon
- "Engineer" → Engineering icon
- "Enchant" → Enchanting icon
- "Fish" → Fishing icon
- "Herb" → Herbalism icon
- "Leather" → Leatherworking icon
- "Mining" → Mining icon
- "Skin" → Skinning icon
- "Tailor" → Tailoring icon
- "Jewelcraft" → Jewelcrafting icon (TBC)

### Comparison with Other Features
This matches how other tracking features work:
- **Auctioneers**: meta.lua includes both meta IDs and real NPC IDs
- **Vendors**: meta.lua includes all vendor NPCs with coordinates
- **Trainers**: Now works the same way after extraction

## Alternative: Manual Addition
If you don't want to set up the full database extraction:

1. Find trainer NPC IDs from a WoW Classic database
2. Add them manually to `db/meta.lua` in the trainer section:
```lua
["trainer"] = {
  -- ... existing entries ...
  [4160] = "AH",  -- Ainethil (Alchemy)
  [1234] = "A",   -- Other trainer (Alliance only)
  -- ... more trainers ...
},
```
3. Ensure the NPC has coordinates in `db/units.lua`
4. The icon system will automatically assign profession icons

## Files Modified
- `toolbox/extractor.lua` - Added trainer flag extraction
- `icons.lua` - Added automatic icon assignment for trainers

## Backwards Compatibility
- Existing functionality unchanged
- Meta IDs still work if they have coordinates
- No breaking changes to configuration or commands
