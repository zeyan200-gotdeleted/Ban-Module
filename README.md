# Ban Module Documentation

[t](TimeBan) 

The Ban module is a Roblox Lua module designed to manage player bans in a Roblox experience. It supports permanent bans, timed bans, and server-specific bans, utilizing Roblox's `DataStoreService` for persistent storage. The module provides methods to check ban status, issue bans, and unban players, with safeguards to prevent banning the experience's creator.

## Contents
1. Overview ~ 20 - 29
2. Set up ~ 31 - 40
3. Module Structure ~ 42 - 79
4. Methods ~ 54 - 55
   - IsBanned ~ 56 - 80
   - PermBan ~ 81 - 100
   - TimeBan ~ 102 - 122
   - ServerBan ~ 124 - 144
   - Unban ~ 145 - 159
5. Events ~ 161 - 204
6. Usage Examples ~ 173 - 204
7. Error Handling ~ 204 - 213


## Overview
- **Purpose**: Manage player bans (permanent, timed, and server-specific) in a Roblox game.
- **Storage**: Uses `DataStoreService` with a DataStore named `"Ban Land"` for persistent ban data.
- **Features**:
  - Check if a player is banned.
  - Issue permanent or timed bans with reasons.
  - Issue server-specific bans (non-persistent).
  - Unban players.
  - Prevent banning the experience creator (user or group owner).
  - Automatically kick banned players on join.

## Setup
1. **Place the Module**:
   - Place the module script in a location accessible to your server scripts (e.g., `ServerScriptService` or `ReplicatedStorage`). (Recommend `ServerScriptService` for safety) 
2. **Require the Module**:
   ```lua
   local Module = require(path.to.BanModule)
   ```
3. **Ensure DataStore Access**:
   - Enable API Services in Roblox Studio (`Game Settings > Security > Enable Studio Access to API Services`).
   - Ensure the game has sufficient DataStore permissions.

## Module Structure
- **Variables**:
  - `DataStore`: The DataStore instance (`"Ban Land"`) for storing ban data.
  - `Callbacks`: Table containing ban-related functions.
  - `Methods`: Empty table (unused in the provided code).
  - `ServerBans`: Table storing UserIds for server-specific bans (non-persistent).
  - `Debounces`: Empty table (unused in the provided code).
  - `Meta`: Metatable returned by the module, linked to `Callbacks`.

- **Metatable**:
  - The module returns `Meta`, which uses `Callbacks` as its `__index` metatable, allowing access to `Callbacks` methods (e.g., `BanModule:IsBanned()`).

## Methods

### IsBanned
**Description**: Checks if a player is banned (permanent or timed).

**Parameters**:
- `Player` (`number` or `string`): The player's UserId or a string representation of it.

**Returns**:
- `boolean`: `true` if the player is banned, `false` otherwise.

**Behavior**:
- Retrieves ban data from the DataStore.
- Handles permanent bans (`BanType = "Perm"`) and timed bans (`BanType = "Time"`).
- For timed bans, checks if the current time (`os.time()`) is less than `UnbanTime`. If expired, automatically unbans the player.
- Returns `false` if no ban data exists or on DataStore failure.

**Example**:
```lua
local isBanned = BanModule:IsBanned(12345678)
if isBanned then
    print("Player is banned.")
else
    print("Player is not banned.")
end
```

### PermBan
**Description**: Permanently bans a player and kicks them with a reason.

**Parameters**:
- `Player` (`Player`): The Player instance to ban.
- `Reason` (`string`): The reason for the ban (displayed on kick).

**Behavior**:
- Checks if the player is the experience creator (user or group owner) and errors if so.
- Updates the DataStore with ban details (`BanType = "Perm"`, `UserId`, `Reason`, `Banned = true`).
- Kicks the player with the provided reason.
- Warns on DataStore failure.

**Example**:
```lua
local player = game.Players:FindFirstChild("PlayerName")
if player then
    BanModule:PermBan(player, "Exploiting")
end
```

### TimeBan
**Description**: Bans a player for a specified duration and kicks them with a reason.

**Parameters**:
- `Player` (`Player`): The Player instance to ban.
- `Reason` (`string`): The reason for the ban (displayed on kick).
- `Duration` (`number`): Ban duration in seconds.

**Behavior**:
- Checks if the player is the experience creator and errors if so.
- Updates the DataStore with ban details (`BanType = "Time"`, `UserId`, `Reason`, `Banned = true`, `UnbanTime`).
- Kicks the player with the provided reason.
- Warns on DataStore failure.

**Example**:
```lua
local player = game.Players:FindFirstChild("PlayerName")
if player then
    BanModule:TimeBan(player, "Toxicity", 3600) -- Ban for 1 hour
end
```

### ServerBan
**Description**: Bans a player from the current server (non-persistent) and kicks them.

**Parameters**:
- `Player` (`Player`): The Player instance to ban.
- `Reason` (`string`): The reason for the ban (displayed on kick).

**Behavior**:
- Checks if the player is the experience creator and errors if so.
- Adds the player's UserId to the `ServerBans` table.
- Kicks the player with the provided reason.
- Does not use DataStore (ban is lost on server restart).

**Example**:
```lua
local player = game.Players:FindFirstChild("PlayerName")
if player then
    BanModule:ServerBan(player, "Disrupting gameplay")
end
```

### UnBan
**Description**: Removes a player's ban from the DataStore.

**Parameters**:
- `Player` (`string`): The player's UserId as a string.

**Behavior**:
- Validates that the input is a string; errors if invalid.
- Removes the player's ban data from the DataStore.
- Warns on DataStore failure.

**Example**:
```lua
BanModule:UnBan("12345678")
```

## Events
The module connects to Roblox signals to enforce bans:

- **Initial Player Check**:
  - On module load, checks all current players in the game.
  - If a player is banned (via `IsBanned`), retrieves their ban reason from the DataStore and kicks them.

- **PlayerAdded**:
  - Listens for new players joining the game.
  - Checks if the player is banned (via `IsBanned`) or server-banned (in `ServerBans`).
  - Kicks banned players with the appropriate reason (DataStore ban reason or "You are banned from this server.").

## Usage Examples
```lua
local BanModule = require(game.ServerScriptService.BanModule)

-- Check if a player is banned
local userId = 12345678
if BanModule:IsBanned(userId) then
    print("Player " .. userId .. " is banned.")
end

-- Permanently ban a player
game.Players.PlayerAdded:Connect(function(player)
    if player.Name == "BadPlayer" then
        Ban:PermBan(player, "Hacking")
    end
end)

-- Ban a player for 24 hours
local player = game.Players:FindFirstChild("SomePlayer")
if player then
    BanModule:TimeBan(player, "Spamming", 24 * 3600)
end

-- Server ban a player
local player = game.Players:FindFirstChild("DisruptivePlayer")
if player then
    BanModule:ServerBan(player, "Causing trouble")
end

-- Unban a player
BanModule:UnBan("12345678")
```

## Error Handling
- **DataStore Failures**:
  - All DataStore operations (`GetAsync`, `UpdateAsync`) are wrapped in `pcall` to handle failures.
  - Failures trigger a `warn` with the error message, and the method returns early or defaults to a safe state (e.g., `IsBanned` returns `false`).
- **Creator Protection**:
  - Attempts to ban the experience creator (user or group owner) result in an error.
- **Invalid Input**:
  - `UnBan` checks for a valid string input and errors if invalid.


