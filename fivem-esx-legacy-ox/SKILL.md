# FiveM ESX Legacy - Ghostli AI Skill

Framework: **ESX Legacy** (`es_extended`) + **ox_lib / ox_target / ox_inventory / oxmysql**

Primary rule: write production-ready, secure, optimized, modern ESX Legacy code based on the **ox** ecosystem as the default standard.

The assistant should behave like a senior FiveM/ESX Lua developer:

* use **oxmysql** for all database work, never old `MySQL.Async.*`,
* use **ox_lib** for notifications, menus, progress bars, zones, callbacks, and inputs,
* use **ox_target** for all in-world interactions, never `DrawMarker` plus distance loops,
* use **ox_inventory** for item registration, usable items, images, and metadata, without `esx_addoninventory` / `esx_inventory`,
* use modern ESX Legacy imports and avoid deprecated patterns,
* validate everything server-side, separate client/server logic, and optimize loops,
* deliver complete resources with `fxmanifest.lua`, explain decisions only when useful, and provide paste-ready code.

## Changes Compared to the Previous Version

* Removed repeated, unnecessary `local ESX = exports['es_extended']:getSharedObject()` and `local ox = require '@ox_lib/init.lua'` from files that already have `@es_extended/imports.lua` and `@ox_lib/init.lua` in the manifest. `ESX` and `lib` are already global there.
* Standardized notifications: always use `lib.notify(...)` / `TriggerClientEvent('ox_lib:notify', ...)` instead of the non-existent `ox.notify(...)` and deprecated `xPlayer.triggerEvent('esx:showNotification', ...)`.
* Fixed the replacement table: `lib.showTextUI`, `lib.progressBar`, `lib.hideContext` / `lib.hideMenu` were previously named incorrectly.
* Added `@ox_lib/init.lua` to `server_scripts` in manifests where the server uses `lib.callback.register`. It was previously missing.
* Fixed the dealership example bug `local vehicle = VEHICLES` to `VEHICLES[model]`.
* Clarified `ox_target:removeZone`. It requires the ID returned by `addBoxZone` / `addSphereZone` / `addPolyZone`, not the option name.
* Added a note about `esx_society` / `esx_addonaccount` as optional, separate dependencies, not part of the core, with `ox_banking` as an alternative.
* Expanded the Hard Bans section and the code review checklist with the points above.

---

## 0. Absolute ESX Initialization Rules

**Preferred modern initialization:** `@es_extended/imports.lua`

For new resources, always add this to `fxmanifest.lua`:

```lua
shared_scripts {
  '@es_extended/imports.lua',
  'config.lua',
}
```

This gives access to the global `ESX` object, and `ESX.PlayerData` on the client side, **without redeclaring** `local ESX = ...` in every file. Similarly, `@ox_lib/init.lua` provides the global `lib` table.

Do not start files with old `ESX = nil` loops.

**Alternative initialization through export**

Use this only when `imports.lua` is not available or when adapting a standalone, single file without a manifest that includes imports:

```lua
local ESX = exports['es_extended']:getSharedObject()
```

**Deprecated initialization - never generate**

```lua
-- ❌ NEVER
ESX = nil

CreateThread(function()
  while ESX == nil do
    TriggerEvent('esx:getSharedObject', function(obj) ESX = obj end)
    Wait(0)
  end
end)
```

```lua
-- ❌ NEVER
TriggerEvent('esx:getSharedObject', function(obj)
  ESX = obj
end)
```

### File-start Templates

**client.lua - modern ESX Legacy start, ox-aware**

```lua
-- ESX and lib are already global because of '@es_extended/imports.lua'
-- and '@ox_lib/init.lua' in fxmanifest.lua. Do not redeclare them locally.

local OxInventory = exports.ox_inventory

local isPlayerLoaded = false
local playerData = {}
local ped = PlayerPedId()

CreateThread(function()
  while not ESX.IsPlayerLoaded() do
    Wait(250)
  end

  isPlayerLoaded = true
  playerData = ESX.GetPlayerData()
  ped = PlayerPedId()
end)

AddEventHandler('esx:playerLoaded', function(xPlayer)
  isPlayerLoaded = true
  playerData = xPlayer
  ped = PlayerPedId()
end)

AddEventHandler('esx:setJob', function(job)
  playerData.job = job
end)

AddEventHandler('esx:playerPedChanged', function(newPed)
  ped = newPed
end)
```

**server.lua - modern ESX Legacy start, oxmysql**

```lua
-- ESX is available globally through '@es_extended/imports.lua' from fxmanifest.lua.
-- Do not use esx:getSharedObject.

local RESOURCE_NAME <const> = GetCurrentResourceName()

AddEventHandler('onResourceStart', function(resourceName)
  if resourceName ~= RESOURCE_NAME then return end

  print(('[%s] started successfully.'):format(RESOURCE_NAME))
end)
```

**config.lua - modern configuration style**

```lua
Config = {}

Config.Debug = false

Config.Locale = 'en'

Config.TargetDistance = 2.0

Config.ProgressBar = {
  duration = 5000,
  label = 'Processing...',
  useWhileDead = false,
  canCancel = true,
  disable = { move = false, car = false, combat = true },
  anim = { dict = 'mini@repair', clip = 'fixing_a_player' },
}
```

---

## 1. Resource Structure

```text
my_resource/
├── fxmanifest.lua
├── config.lua
├── client.lua
├── server.lua
└── locales/
    └── en.lua
```

Optional extended structure:

```text
my_resource/
├── fxmanifest.lua
├── shared/
│   ├── config.lua
│   └── utils.lua
├── client/
│   ├── main.lua
│   ├── targets.lua
│   └── ui.lua
├── server/
│   ├── main.lua
│   ├── callbacks.lua
│   └── database.lua
└── locales/
    ├── pl.lua
    └── en.lua
```

---

## 2. Modern fxmanifest.lua

> **Important:** if the server uses `lib.callback.register`, `lib.callback.await`, or any other server-side `lib.*` function, `@ox_lib/init.lua` must be included **both** in `client_scripts` **and** in `server_scripts`, or in `shared_scripts`.

**Minimal recommended manifest, ox ecosystem**

```lua
fx_version 'cerulean'
game 'gta5'

description 'Modern ESX Legacy resource (ox-powered)'
author 'YourName'
version '1.0.0'

lua54 'yes'

shared_scripts {
  '@es_extended/imports.lua',
  'config.lua',
}

client_scripts {
  '@ox_lib/init.lua',
  'client.lua',
}

server_scripts {
  '@oxmysql/lib/MySQL.lua',
  '@ox_lib/init.lua',
  'server.lua',
}

dependencies {
  'es_extended',
  'oxmysql',
  'ox_lib',
  'ox_inventory',
  'ox_target',
}
```

**With localization support**

```lua
fx_version 'cerulean'
game 'gta5'

description 'Modern ESX Legacy resource (ox-powered)'
author 'YourName'
version '1.0.0'

lua54 'yes'

shared_scripts {
  '@es_extended/imports.lua',
  '@es_extended/locale.lua',
  'locales/*.lua',
  'config.lua',
}

client_scripts {
  '@ox_lib/init.lua',
  'client.lua',
}

server_scripts {
  '@oxmysql/lib/MySQL.lua',
  '@ox_lib/init.lua',
  'server.lua',
}

dependencies {
  'es_extended',
  'oxmysql',
  'ox_lib',
  'ox_inventory',
  'ox_target',
}
```

**With multiple folders**

```lua
fx_version 'cerulean'
game 'gta5'

description 'Advanced ESX Legacy resource (ox-powered)'
author 'YourName'
version '1.0.0'

lua54 'yes'

shared_scripts {
  '@es_extended/imports.lua',
  '@ox_lib/init.lua',
  'shared/*.lua',
}

client_scripts {
  'client/*.lua',
}

server_scripts {
  '@oxmysql/lib/MySQL.lua',
  'server/*.lua',
}

dependencies {
  'es_extended',
  'oxmysql',
  'ox_lib',
  'ox_inventory',
  'ox_target',
}
```

---

## 3. Lua 5.4 Coding Rules

Always enable:

```lua
lua54 'yes'
```

Use Lua 5.4 style where it makes sense:

```lua
local MAX_DISTANCE <const> = 3.0
local RESOURCE_NAME <const> = GetCurrentResourceName()
```

Use `local` by default:

```lua
-- ❌ Bad
playerData = ESX.GetPlayerData()

-- ✅ Good
local playerData = ESX.GetPlayerData()
```

Avoid unnecessary global variables. Global values should only be used for framework exports or configuration tables such as `Config`.

**Naming**

```lua
-- Local variables and functions
local playerData = {}
local function openMenu() end

-- Constants
local MAX_DISTANCE <const> = 3.0

-- Resource events
local EVENT_OPEN_MENU <const> = 'my_resource:openMenu'
```

**Prefer early returns**

```lua
RegisterNetEvent('my_resource:server:pay', function(amount)
  local src = source
  local xPlayer = ESX.Player(src)

  if not xPlayer then return end
  if type(amount) ~= 'number' then return end
  if amount <= 0 then return end

  xPlayer.addMoney(amount)
end)
```

---

## 4. Modern FiveM Native Patterns

Use modern natives and Lua vector math.

| Deprecated / weak                 | Modern / preferred                                            |
| --------------------------------- | ------------------------------------------------------------- |
| `GetPlayerPed(-1)`                | `PlayerPedId()`                                               |
| `GetPlayerPedId(-1)`              | `PlayerPedId()`                                               |
| `GetDistanceBetweenCoords(...)`   | `#(coordsA - coordsB)`                                        |
| `GetHashKey('adder')`             | `joaat('adder')`                                              |
| `table.insert(t, v)`              | `t[#t + 1] = v`                                               |
| `ESX.UI.Menu` for menus and shops | `lib.registerContext` / `lib.registerMenu` (ox_lib)           |
| `ESX.ShowNotification`            | `lib.notify`                                                  |
| `ESX.TextUI`                      | `lib.showTextUI`                                              |
| `ESX.HideUI`                      | `lib.hideTextUI`                                              |
| `ESX.Progressbar`                 | `lib.progressBar` / `lib.progressCircle`                      |
| `ESX.UI.Menu.CloseAll`            | `lib.hideContext` / `lib.hideMenu`                            |
| `DrawMarker` plus distance loop   | `exports.ox_target:addSphereZone` / `:addBoxZone`             |
| `ESX.GetPlayerInventory`          | `exports.ox_inventory:GetPlayerItems`                         |
| `xPlayer.getInventoryItem(name)`  | `exports.ox_inventory:GetItemCount(src, name)`                |
| `xPlayer.addInventoryItem`        | `exports.ox_inventory:AddItem(src, name, count, metadata)`    |
| `xPlayer.removeInventoryItem`     | `exports.ox_inventory:RemoveItem(src, name, count, metadata)` |
| `MySQL.Async.fetchAll`            | `MySQL.query.await`                                           |
| `MySQL.Async.fetchScalar`         | `MySQL.scalar.await`                                          |
| `MySQL.Async.execute`             | `MySQL.update.await` / `MySQL.insert.await`                   |

Example:

```lua
local ped = PlayerPedId()
local coords = GetEntityCoords(ped)
local target = vector3(215.76, -810.12, 30.73)

if #(coords - target) <= 2.0 then
  lib.showTextUI('Press [E] to open the menu')
end
```

---

## 5. ESX Client Patterns

**Wait for the player to load**

```lua
CreateThread(function()
  while not ESX.IsPlayerLoaded() do
    Wait(250)
  end

  local playerData = ESX.GetPlayerData()
  print(('Loaded as job: %s'):format(playerData.job.name))
end)
```

**Update PlayerData**

```lua
local playerData = {}

AddEventHandler('esx:playerLoaded', function(xPlayer)
  playerData = xPlayer
end)

AddEventHandler('esx:setJob', function(job)
  playerData.job = job
end)
```

**Secure client event from server only**

Use `ESX.SecureNetEvent` when an event should only be triggered by the server.

```lua
ESX.SecureNetEvent('my_resource:client:notify', function(message)
  lib.notify({ description = message, type = 'inform' })
end)
```

**Client command**

There is no `ESX.RegisterCommand` on the client side. Use the native `RegisterCommand`.

```lua
RegisterCommand('myclientcmd', function()
  if not ESX.IsPlayerLoaded() then return end

  lib.notify({ description = 'Client command executed.', type = 'success' })
end, false)
```

---

## 6. ESX Server Patterns

**Get a single player**

Preferred style:

```lua
local xPlayer = ESX.Player(source)
if not xPlayer then return end
```

Fallback only when backward compatibility is needed:

```lua
local xPlayer = ESX.GetPlayerFromId(source)
```

Do not use the fallback if the user does not explicitly need support for older ESX versions.

**Get players with a filter**

```lua
local policePlayers = ESX.ExtendedPlayers('job', 'police')

for _, xPlayer in ipairs(policePlayers) do
  TriggerClientEvent('ox_lib:notify', xPlayer.source, {
    description = 'Dispatch: new alert.',
    type = 'inform',
  })
end
```

> Note: `xPlayer.triggerEvent('esx:showNotification', ...)` still triggers the old ESX notification flow on the client side. If you use ox_lib, send `TriggerClientEvent('ox_lib:notify', xPlayer.source, { description = '...', type = 'inform' })` instead.

**Grouped players**

```lua
local groupedPlayers = ESX.ExtendedPlayers('job', { 'police', 'ambulance' })

for jobName, players in pairs(groupedPlayers) do
  for _, xPlayer in ipairs(players) do
    print(jobName, xPlayer.getName())
  end
end
```

**All players**

```lua
local players = ESX.ExtendedPlayers()

for _, xPlayer in ipairs(players) do
  print(xPlayer.source, xPlayer.getName())
end
```

---

## 7. xPlayer Rules

**Most common xPlayer methods** still supported, but prefer ox_inventory for item operations.

```lua
xPlayer.getName()
xPlayer.getIdentifier()
xPlayer.getJob()
xPlayer.setJob('police', 2)

xPlayer.getMoney()
xPlayer.addMoney(500)
xPlayer.removeMoney(500)

xPlayer.getAccount('bank')
xPlayer.addAccountMoney('bank', 1000)
xPlayer.removeAccountMoney('bank', 1000)

-- xPlayer.getInventoryItem / addInventoryItem / removeInventoryItem:
-- only as a fallback for older dependencies when ox_inventory is not available
xPlayer.canRemoveInventoryItem('bread', 1)

xPlayer.triggerEvent('event:name', ...)
```

**Always validate xPlayer**

```lua
RegisterNetEvent('my_resource:server:doSomething', function()
  local src = source
  local xPlayer = ESX.Player(src)

  if not xPlayer then
    return
  end

  -- Safe logic here
end)
```

**Never trust xPlayer data sent by the client**

```lua
-- ❌ Bad
RegisterNetEvent('shop:buy', function(price)
  local xPlayer = ESX.Player(source)
  xPlayer.removeMoney(price)
end)
```

```lua
-- ✅ Good
local ITEMS <const> = {
  bread = { price = 50, item = 'bread' },
  water = { price = 35, item = 'water' },
}

RegisterNetEvent('shop:buy', function(itemName)
  local src = source
  local xPlayer = ESX.Player(src)
  if not xPlayer then return end

  local item = ITEMS[itemName]
  if not item then return end

  if xPlayer.getMoney() < item.price then
    return TriggerClientEvent('ox_lib:notify', src, {
      description = 'You do not have enough cash.',
      type = 'error',
    })
  end

  xPlayer.removeMoney(item.price)
  exports.ox_inventory:AddItem(src, item.item, 1)

  TriggerClientEvent('ox_lib:notify', src, {
    description = ('Purchased: %s'):format(item.item),
    type = 'success',
  })
end)
```

---

## 8. Commands

**Server command with `ESX.RegisterCommand`**

```lua
ESX.RegisterCommand('givemoney', 'admin', function(xPlayer, args, showError)
  local targetPlayer = ESX.Player(args.target)
  if not targetPlayer then
    return showError('Player does not exist.')
  end

  local amount = args.amount
  if amount <= 0 then
    return showError('Amount must be greater than 0.')
  end

  targetPlayer.addMoney(amount)

  TriggerClientEvent('ox_lib:notify', xPlayer.source, {
    description = ('You gave $%s to %s.'):format(amount, targetPlayer.getName()),
    type = 'success',
  })
  TriggerClientEvent('ox_lib:notify', targetPlayer.source, {
    description = ('You received $%s.'):format(amount),
    type = 'success',
  })
end, true, {
  help = 'Give cash to a player',
  validate = true,
  arguments = {
    { name = 'target', help = 'Player ID', type = 'playerId' },
    { name = 'amount', help = 'Amount',    type = 'number'   },
  },
})
```

**Multiple admin groups**

```lua
ESX.RegisterCommand('adminnotify', { 'admin', 'superadmin' }, function(xPlayer, args, showError)
  local message = table.concat(args.message, ' ')

  if message == '' then
    return showError('Enter a message.')
  end

  TriggerClientEvent('ox_lib:notify', -1, { description = message, type = 'inform' })
end, true, {
  help = 'Send an admin announcement',
  validate = true,
  arguments = {
    { name = 'message', help = 'Message content', type = 'string' },
  },
})
```

---

## 9. Callbacks

**Preferred: ox_lib callbacks**

For new code, use `lib.callback` by default. This requires `@ox_lib/init.lua` in `server_scripts`, see section 2.

```lua
-- server
lib.callback.register('my_resource:getPlayerInfo', function(source)
  local xPlayer = ESX.Player(source)
  if not xPlayer then return nil end

  return {
    name = xPlayer.getName(),
    identifier = xPlayer.getIdentifier(),
    job = xPlayer.getJob(),
    money = xPlayer.getMoney(),
  }
end)
```

```lua
-- client
lib.callback('my_resource:getPlayerInfo', false, function(data)
  if not data then
    return lib.notify({ description = 'Failed to fetch data.', type = 'error' })
  end

  print(data.name, data.job.name)
end)
```

Server-side `await` version, rarely needed but available:

```lua
local coords = lib.callback.await('my_resource:client:getVehicleCoords', source)
```

Use server to client callbacks only for non-critical information, such as vehicle position for preview purposes. Never use them for security validation.

**Alternative for backward compatibility: `ESX.RegisterServerCallback`**

If the resource must remain compatible with older dependencies without ox_lib, you can use:

```lua
ESX.RegisterServerCallback('my_resource:getPlayerInfo', function(src, cb)
  local xPlayer = ESX.Player(src)
  if not xPlayer then return cb(nil) end

  cb({
    name = xPlayer.getName(),
    identifier = xPlayer.getIdentifier(),
    job = xPlayer.getJob(),
    money = xPlayer.getMoney(),
  })
end)
```

Do not mix both styles in one resource without a reason. Choose one, preferably ox_lib, and stay consistent.

---

## 10. Items, ox_inventory Preferred

**Register a usable item with ox_inventory**

```lua
exports.ox_inventory:RegisterUse('bread', function(source)
  local src = source

  if exports.ox_inventory:RemoveItem(src, 'bread', 1) then
    TriggerClientEvent('ox_lib:notify', src, { description = 'You ate bread.', type = 'success' })
  end
end)
```

**Register an item with metadata, ox_inventory**

Item definitions are located in `ox_inventory/data/items.lua`. This is ox_inventory configuration, not your resource:

```lua
-- in ox_inventory/data/items.lua
['energy_drink'] = {
  label = 'Energy Drink',
  weight = 1,
  stack = true,
  close = true,
  description = 'Red Bull gives you wings.',
  client = { image = 'energy_drink.png' },
},

['gold_bar'] = {
  label = 'Gold Bar',
  weight = 5,
  stack = true,
  close = true,
  description = 'A heavy gold bar.',
  client = { image = 'gold_bar.png' },
},
```

**Check inventory server-side, ox_inventory**

```lua
local count = exports.ox_inventory:GetItemCount(source, 'bread', nil, false)

if count and count >= 1 then
  exports.ox_inventory:RemoveItem(source, 'bread', 1)
end
```

**Check inventory client-side, only for UI/UX**

```lua
local count = exports.ox_inventory:GetItemCount('bread', nil, false)

if count and count > 0 then
  lib.notify({ description = ('You have %sx bread.'):format(count), type = 'inform' })
end
```

**Open player inventory, UI**

```lua
-- client
RegisterCommand('inv', function()
  exports.ox_inventory:openInventory('player', cache.playerId or GetPlayerServerId(PlayerId()))
end, false)
```

---

## 11. Money and Accounts

**Cash**

```lua
local money = xPlayer.getMoney()

if money < 500 then
  return TriggerClientEvent('ox_lib:notify', xPlayer.source, {
    description = 'You do not have enough cash.',
    type = 'error',
  })
end

xPlayer.removeMoney(500)
```

**Bank**

```lua
local bank = xPlayer.getAccount('bank')

if not bank or bank.money < 1000 then
  return TriggerClientEvent('ox_lib:notify', xPlayer.source, {
    description = 'You do not have enough money in your bank account.',
    type = 'error',
  })
end

xPlayer.removeAccountMoney('bank', 1000)
```

**Society money, esx_society as an optional separate dependency**

`esx_society` / `esx_addonaccount` **are not part of the `es_extended` core**. They are separate, additional resources. Many new servers no longer use them and replace them with solutions like `ox_banking` or a custom company account system. Use the pattern below only if `esx_society` and `esx_addonaccount` are actually installed:

```lua
TriggerEvent('esx_society:getSociety', 'police', function(society)
  if not society then return end
  TriggerEvent('esx_addonaccount:getSharedAccount', 'society_' .. society.name, function(account)
    account.addMoney(5000)
  end)
end)
```

**Never blindly accept an amount from the client**

```lua
-- ✅ Server defines prices
local PRICES <const> = {
  repair = 750,
  wash = 100,
}
```

---

## 12. Jobs

**Check job on client**

```lua
local function isPolice()
  return ESX.PlayerData.job and ESX.PlayerData.job.name == 'police'
end
```

**Check job on server**

```lua
local function hasJob(xPlayer, jobName)
  local job = xPlayer.getJob()
  return job and job.name == jobName
end
```

**Check grade**

```lua
local function hasMinimumGrade(xPlayer, jobName, minGrade)
  local job = xPlayer.getJob()
  return job and job.name == jobName and job.grade >= minGrade
end
```

**Job-restricted event**

```lua
RegisterNetEvent('police_resource:server:openArmory', function()
  local src = source
  local xPlayer = ESX.Player(src)
  if not xPlayer then return end

  local job = xPlayer.getJob()
  if not job or job.name ~= 'police' then
    return TriggerClientEvent('ox_lib:notify', src, {
      description = 'You do not have access.',
      type = 'error',
    })
  end

  xPlayer.triggerEvent('police_resource:client:openArmory')
end)
```

---

## 13. ox_target - Default In-world Interaction System

**Rule:** use `ox_target` for all in-world interactions. Never use `DrawMarker` plus distance loop plus `IsControlJustPressed(0, 38)` for primary interactions.

**Box zone, static point**

```lua
-- client
local zoneId = exports.ox_target:addBoxZone({
  coords = vec3(25.74, -1347.33, 29.5),
  size   = vec3(1.5, 1.5, 2.0),
  rotation = 0.0,
  debug   = Config.Debug,
  options = {
    {
      name   = 'example_shop:open',
      label  = 'Open shop',
      icon   = 'fa-solid fa-shop',
      groups = { 'police' }, -- optional job restriction
      onSelect = function()
        openShop()
      end,
    },
  },
  distance = 2.0,
})
```

**Sphere zone**

```lua
local zoneId = exports.ox_target:addSphereZone({
  coords = vec3(100.0, 100.0, 30.0),
  radius = 2.0,
  options = {
    {
      name = 'drug_seller:sell',
      label = 'Sell goods',
      icon = 'fa-solid fa-hand-holding-dollar',
      onSelect = function()
        TriggerServerEvent('drug:server:sell')
      end,
    },
  },
})
```

**Ped target, NPC**

```lua
exports.ox_target:addLocalEntity(ped, {
  {
    name = 'mechanic:talk',
    label = 'Talk to the mechanic',
    icon = 'fa-solid fa-wrench',
    onSelect = function()
      openMechanicMenu()
    end,
  },
})
```

**Networked entity target, vehicle or object**

```lua
exports.ox_target:addGlobalVehicle({
  {
    name = 'police:checkPlate',
    label = 'Check plate',
    icon = 'fa-solid fa-car',
    job   = 'police',
    onSelect = function(data)
      TriggerServerEvent('police:server:checkPlate', data.entity)
    end,
  },
})
```

**Removing a zone**

`removeZone` accepts the **zone ID returned by `addBoxZone` / `addSphereZone` / `addPolyZone`**, not the option `name`. Save the ID when creating the zone:

```lua
local zoneId = exports.ox_target:addBoxZone({ ... })

-- Later, when the zone is no longer needed:
exports.ox_target:removeZone(zoneId)
```

Entities such as `addGlobalVehicle`, `addLocalEntity`, and similar are removed using the matching `removeGlobalVehicle('name')` / `removeLocalEntity(entity, 'name')`, where `'name'` is the specific option name.

**Notes about loops**

Remove all interaction loops based on `DrawMarker` plus `Wait(0)`. Replace them with `ox_target` zones. Use `lib.zones` / `lib.points` only for visual effects such as text UI, particles, or area checks. Do not use them for primary interactions when `ox_target` is available.

---

## 14. ox_lib UI Patterns

**Notifications, instead of `ESX.ShowNotification`**

```lua
-- client
lib.notify({
  title    = 'Shop',
  description = 'Purchase successful',
  type     = 'success', -- 'inform' | 'error' | 'warning' | 'success'
  position = 'top-right',
  duration = 4000,
})
```

```lua
-- server -> client
TriggerClientEvent('ox_lib:notify', source, {
  title = 'Bank',
  description = 'Deposit accepted.',
  type = 'success',
})
```

**TextUI, instead of `ESX.TextUI` / `ESX.HideUI`**

```lua
lib.showTextUI('Press [E] to enter', {
  position = 'left-center',
  icon = 'hand',
})

lib.hideTextUI()
```

Track state so you do not call it every frame:

```lua
local showingTextUI = false

local function showTextUI(message, options)
  if showingTextUI then return end
  showingTextUI = true
  lib.showTextUI(message, options)
end

local function hideTextUI()
  if not showingTextUI then return end
  showingTextUI = false
  lib.hideTextUI()
end
```

**Context menu, instead of `ESX.UI.Menu` for small lists**

```lua
lib.registerContext({
  id = 'example_shop:menu',
  title = 'Shop',
  options = {
    { title = 'Bread - $50',  description = 'Buy bread',        icon = 'bread-slice', event = 'example_shop:client:buy', args = { item = 'bread'  } },
    { title = 'Water - $35',  description = 'Buy water',        icon = 'glass-water', event = 'example_shop:client:buy', args = { item = 'water'  } },
    { title = 'Energy - $80', description = 'Buy energy drink', icon = 'bolt',        event = 'example_shop:client:buy', args = { item = 'energy_drink' } },
  },
})

local function openShop()
  lib.showContext('example_shop:menu')
end

AddEventHandler('example_shop:client:buy', function(data)
  TriggerServerEvent('example_shop:server:buyItem', data.item)
end)
```

**Menu builder with submenu support**

```lua
lib.registerMenu({
  id = 'mechanic:main',
  title = 'Mechanic',
  position = 'top-right',
  options = {
    { label = 'Repair vehicle', description = '$750', args = { action = 'repair' } },
    { label = 'Wash vehicle',   description = '$150', args = { action = 'wash'   } },
  },
}, function(selected, scrollIndex, args)
  if args.action == 'repair' then
    TriggerServerEvent('mechanic:server:repair')
  end
end)

local function openMechanicMenu()
  lib.showMenu('mechanic:main')
end
```

**Progress bar, instead of `ESX.Progressbar`**

```lua
local function doProgress(label, duration, onComplete)
  if lib.progressBar({
    duration    = duration,
    label       = label,
    useWhileDead = false,
    canCancel   = true,
    disable     = { move = false, car = true, combat = true },
    anim        = { dict = 'mini@repair', clip = 'fixing_a_player' },
  }) then
    onComplete()
  else
    lib.notify({ description = 'Cancelled.', type = 'error' })
  end
end
```

**Input dialog, instead of `ESX.UI.Menu.Open` with `'dialog'`**

```lua
local input = lib.inputDialog('Enter amount', {
  { type = 'number', label = 'How much?', min = 1, max = 100000, required = true },
})

if not input then return end

TriggerServerEvent('bank:server:deposit', input[1])
```

---

## 15. Vehicles

**Spawn a vehicle client-side**

```lua
local function spawnVehicle(model, coords, heading)
  local modelHash = joaat(model)

  RequestModel(modelHash)
  while not HasModelLoaded(modelHash) do
    Wait(0)
  end

  local vehicle = CreateVehicle(modelHash, coords.x, coords.y, coords.z, heading, true, false)

  SetVehicleOnGroundProperly(vehicle)
  SetPedIntoVehicle(PlayerPedId(), vehicle, -1)
  SetModelAsNoLongerNeeded(modelHash)

  return vehicle
end
```

**ox_target vehicle interaction**

```lua
exports.ox_target:addGlobalVehicle({
  {
    name  = 'mechanic:repair',
    label = 'Repair vehicle',
    icon  = 'fa-solid fa-screwdriver-wrench',
    job   = 'mechanic',
    onSelect = function(data)
      lib.callback('mechanic:client:canRepair', false, function(can)
        if not can then return end
        doProgress('Repairing vehicle...', 8000, function()
          TriggerServerEvent('mechanic:server:repair', VehToNet(data.entity))
        end)
      end)
    end,
  },
})
```

**The server must validate vehicle purchases**

```lua
local VEHICLES <const> = {
  adder  = { label = 'Adder',  price = 1000000 },
  sultan = { label = 'Sultan', price = 25000   },
}

RegisterNetEvent('dealership:server:buyVehicle', function(model)
  local src = source
  local xPlayer = ESX.Player(src)
  if not xPlayer then return end

  local vehicle = VEHICLES[model]
  if not vehicle then return end

  local bank = xPlayer.getAccount('bank')
  if not bank or bank.money < vehicle.price then
    return TriggerClientEvent('ox_lib:notify', src, { description = 'You do not have enough money.', type = 'error' })
  end

  xPlayer.removeAccountMoney('bank', vehicle.price)
  xPlayer.triggerEvent('dealership:client:spawnPurchasedVehicle', model)
end)
```

---

## 16. Database with oxmysql

Use `oxmysql` for all database work. All queries are async/await.

**Async query**

```lua
local result = MySQL.query.await(
  'SELECT * FROM owned_vehicles WHERE owner = ?',
  { xPlayer.getIdentifier() }
)
```

**Insert**

```lua
local id = MySQL.insert.await(
  'INSERT INTO my_table (identifier, value) VALUES (?, ?)',
  { xPlayer.getIdentifier(), 100 }
)
```

**Update**

```lua
local affected = MySQL.update.await(
  'UPDATE my_table SET value = ? WHERE identifier = ?',
  { 200, xPlayer.getIdentifier() }
)
```

**Scalar**

```lua
local count = MySQL.scalar.await(
  'SELECT COUNT(*) FROM my_table WHERE identifier = ?',
  { xPlayer.getIdentifier() }
)
```

**Multiple queries, transaction**

```lua
MySQL.transaction.await({
  { query = 'UPDATE users SET money = money - ? WHERE identifier = ?', values = { 500, src_id } },
  { query = 'UPDATE users SET money = money + ? WHERE identifier = ?', values = { 500, dst_id } },
})
```

**Database security**

Always use parameter binding:

```lua
-- ✅ Good
MySQL.query.await('SELECT * FROM users WHERE identifier = ?', { identifier })
```

Never concatenate user data into SQL:

```lua
-- ❌ Bad
MySQL.query.await('SELECT * FROM users WHERE identifier = "' .. identifier .. '"')
```

---

## 17. Event Naming Convention

Use clear, namespaced event names:

```lua
'resource_name:client:openMenu'
'resource_name:client:notify'
'resource_name:server:buyItem'
'resource_name:server:saveData'
```

Examples:

```lua
RegisterNetEvent('my_shop:server:buyItem', function(itemName)
  -- server logic
end)

ESX.SecureNetEvent('my_shop:client:openMenu', function(items)
  -- client logic
end)
```

**Rules:** `client` events are handled client-side, `server` events are handled server-side. The server must validate every important action. Client events triggered by the server should use `ESX.SecureNetEvent` when appropriate.

---

## 18. Security Rules

**Never trust the client with:**

money amounts, item counts, item prices, job permissions, grade permissions, admin permissions, vehicle ownership, database identifiers, cooldown bypasses, distance checks for rewards, or society money changes.

**Always validate server-side**

```lua
RegisterNetEvent('drug:server:sell', function()
  local src = source
  local xPlayer = ESX.Player(src)
  if not xPlayer then return end

  local ped = GetPlayerPed(src)
  local coords = GetEntityCoords(ped)
  local sellPoint = vector3(100.0, 100.0, 30.0)

  if #(coords - sellPoint) > 5.0 then
    return
  end

  if exports.ox_inventory:GetItemCount(src, 'weed', nil, false) < 1 then
    return TriggerClientEvent('ox_lib:notify', src, { description = 'You do not have the goods.', type = 'error' })
  end

  exports.ox_inventory:RemoveItem(src, 'weed', 1)
  xPlayer.addMoney(250)
end)
```

**Add cooldowns to exploit-prone events**

```lua
local cooldowns = {}

local function isOnCooldown(src, seconds)
  local now = os.time()
  local last = cooldowns[src] or 0

  if now - last < seconds then
    return true
  end

  cooldowns[src] = now
  return false
end

RegisterNetEvent('my_resource:server:reward', function()
  local src = source

  if isOnCooldown(src, 5) then
    return
  end

  local xPlayer = ESX.Player(src)
  if not xPlayer then return end

  xPlayer.addMoney(100)
end)

AddEventHandler('playerDropped', function()
  cooldowns[source] = nil
end)
```

---

## 19. Discord Logs

> Requires a configured Discord webhook in ESX convars, such as `setr esx:discordWebhook`. Logging functions will not work without server-side configuration.

**Simple log**

```lua
ESX.DiscordLog('UserActions', 'Player Joined', 'green', 'A new player joined the server.')
```

**Log with fields**

```lua
ESX.DiscordLogFields('UserActions', 'Item Purchase', 'green', {
  { name = 'Player',     value = xPlayer.getName(),       inline = true },
  { name = 'Identifier', value = xPlayer.getIdentifier(), inline = true },
  { name = 'Item',       value = itemName,                inline = true },
})
```

Do not log secrets such as database passwords, webhooks, tokens, or private identifiers, unless the user explicitly needs audit logs.

---

## 20. Complete Modern Resource Example, ox-powered

**fxmanifest.lua**

```lua
fx_version 'cerulean'
game 'gta5'

description 'Example modern ESX Legacy shop (ox-powered)'
author 'YourName'
version '1.0.0'

lua54 'yes'

shared_scripts {
  '@es_extended/imports.lua',
  'config.lua',
}

client_scripts {
  '@ox_lib/init.lua',
  'client.lua',
}

server_scripts {
  '@oxmysql/lib/MySQL.lua',
  '@ox_lib/init.lua',
  'server.lua',
}

dependencies {
  'es_extended',
  'oxmysql',
  'ox_lib',
  'ox_inventory',
  'ox_target',
}
```

**config.lua**

```lua
Config = {}

Config.Debug = false

Config.Shop = {
  Coords      = vec3(25.74, -1347.33, 29.5),
  Size        = vec3(1.5, 1.5, 2.0),
  Rotation    = 0.0,
  Distance    = 2.0,
  JobAccess   = { 'police', 'ambulance' }, -- nil = no restriction
}

Config.Items = {
  bread        = { label = 'Bread',        price = 50  },
  water        = { label = 'Water',        price = 35  },
  energy_drink = { label = 'Energy Drink', price = 80  },
}
```

**client.lua**

```lua
-- ESX and lib are global because of '@es_extended/imports.lua' and '@ox_lib/init.lua'
local OxInv = exports.ox_inventory

local isPlayerLoaded = false
local showingTextUI  = false

local function showTextUI(message, options)
  if showingTextUI then return end
  showingTextUI = true
  lib.showTextUI(message, options or { position = 'left-center', icon = 'shop' })
end

local function hideTextUI()
  if not showingTextUI then return end
  showingTextUI = false
  lib.hideTextUI()
end

local function buildShopMenu()
  local options = {}

  for itemName, itemData in pairs(Config.Items) do
    options[#options + 1] = {
      title       = ('%s - $%s'):format(itemData.label, itemData.price),
      description = 'Click to buy',
      icon        = 'shopping-bag',
      onSelect    = function()
        lib.callback('example_shop:client:canBuy', false, function(canBuy, reason)
          if canBuy then
            TriggerServerEvent('example_shop:server:buyItem', itemName)
          else
            lib.notify({ description = reason or 'You cannot buy this.', type = 'error' })
          end
        end, itemName)
      end,
    }
  end

  lib.registerContext({
    id      = 'example_shop:menu',
    title   = 'Shop',
    options = options,
  })
end

CreateThread(function()
  while not ESX.IsPlayerLoaded() do
    Wait(250)
  end
  isPlayerLoaded = true
end)

AddEventHandler('esx:playerLoaded', function()
  isPlayerLoaded = true
end)

CreateThread(function()
  Wait(500)
  buildShopMenu()

  local options = {
    {
      name      = 'example_shop:open',
      label     = 'Open shop',
      icon      = 'fa-solid fa-shop',
      onSelect  = function()
        if not isPlayerLoaded then return end
        lib.showContext('example_shop:menu')
      end,
    },
  }

  if Config.Shop.JobAccess and #Config.Shop.JobAccess > 0 then
    options[1].groups = Config.Shop.JobAccess
  end

  exports.ox_target:addBoxZone({
    coords    = Config.Shop.Coords,
    size      = Config.Shop.Size,
    rotation  = Config.Shop.Rotation,
    debug     = Config.Debug,
    options   = options,
    distance  = Config.Shop.Distance,
  })
end)
```

**server.lua**

```lua
-- ESX and lib are global because of '@es_extended/imports.lua' and '@ox_lib/init.lua'
local OxInv = exports.ox_inventory

RegisterNetEvent('example_shop:server:buyItem', function(itemName)
  local src = source
  local xPlayer = ESX.Player(src)
  if not xPlayer then return end

  local itemData = Config.Items[itemName]
  if not itemData then return end

  if xPlayer.getMoney() < itemData.price then
    return TriggerClientEvent('ox_lib:notify', src, {
      description = 'You do not have enough cash.',
      type = 'error',
    })
  end

  if not OxInv:CanAddItem(src, itemName, 1) then
    return TriggerClientEvent('ox_lib:notify', src, {
      description = 'Not enough inventory space.',
      type = 'error',
    })
  end

  xPlayer.removeMoney(itemData.price)

  if not OxInv:AddItem(src, itemName, 1) then
    xPlayer.addMoney(itemData.price) -- rollback
    return TriggerClientEvent('ox_lib:notify', src, {
      description = 'Failed to add item.',
      type = 'error',
    })
  end

  TriggerClientEvent('ox_lib:notify', src, {
    description = ('Purchased: %s'):format(itemData.label),
    type = 'success',
  })
end)

lib.callback.register('example_shop:client:canBuy', function(source, itemName)
  local xPlayer = ESX.Player(source)
  if not xPlayer then return false, 'You are not logged in.' end

  local itemData = Config.Items[itemName]
  if not itemData then return false, 'Unknown item.' end
  if xPlayer.getMoney() < itemData.price then return false, 'You do not have enough cash.' end
  if not OxInv:CanAddItem(source, itemName, 1) then return false, 'Not enough inventory space.' end

  return true
end)
```

---

## 21. Modernization Rules for Old ESX Scripts

When the user provides old code, modernize it automatically to the ox ecosystem.

**Old ESX object loading**

```lua
-- ❌ Old
ESX = nil
TriggerEvent('esx:getSharedObject', function(obj) ESX = obj end)
```

```lua
-- ✅ New
-- Use '@es_extended/imports.lua' in fxmanifest.lua. ESX is global then.
-- Or, if imports are not possible:
local ESX = exports['es_extended']:getSharedObject()
```

**Old xPlayer access**

```lua
-- ❌ Old / legacy
local xPlayer = ESX.GetPlayerFromId(source)
```

```lua
-- ✅ Preferred
local xPlayer = ESX.Player(source)
```

**Old player loops**

```lua
-- ❌ Old / legacy
local players = ESX.GetPlayers()
for i = 1, #players do
  local xPlayer = ESX.GetPlayerFromId(players[i])
end
```

```lua
-- ✅ Preferred
local players = ESX.ExtendedPlayers()
for _, xPlayer in ipairs(players) do
  print(xPlayer.getName())
end
```

**Old job filtering**

```lua
-- ❌ Old
for _, playerId in ipairs(ESX.GetPlayers()) do
  local xPlayer = ESX.GetPlayerFromId(playerId)
  if xPlayer.job.name == 'police' then
    -- logic
  end
end
```

```lua
-- ✅ New
local policePlayers = ESX.ExtendedPlayers('job', 'police')
for _, xPlayer in ipairs(policePlayers) do
  -- logic
end
```

**`ESX.UI.Menu` to ox_lib context/menu**

```lua
-- ❌ Old
ESX.UI.Menu.Open('default', GetCurrentResourceName(), 'shop_menu', {
  title = 'Shop',
  align = 'top-left',
  elements = { { label = 'Bread - $50', value = 'bread' } },
}, function(data, menu)
  TriggerServerEvent('shop:buy', data.current.value)
  menu.close()
end, function(data, menu) menu.close() end)
```

```lua
-- ✅ New
lib.registerContext({
  id = 'shop:menu',
  title = 'Shop',
  options = {
    { title = 'Bread - $50', onSelect = function()
        TriggerServerEvent('shop:buy', 'bread')
    end },
  },
})
lib.showContext('shop:menu')
```

**`ESX.ShowNotification` / `xPlayer.triggerEvent('esx:showNotification', ...)` to `lib.notify`**

```lua
-- ❌ Old
ESX.ShowNotification('Message')
-- or
xPlayer.triggerEvent('esx:showNotification', 'Message')
TriggerClientEvent('esx:showNotification', source, 'Message')
```

```lua
-- ✅ New
-- client
lib.notify({ description = 'Message', type = 'inform' })

-- server -> client
TriggerClientEvent('ox_lib:notify', src, { description = 'Message', type = 'inform' })
```

**`ESX.TextUI` to `lib.showTextUI`**

```lua
-- ❌ Old
ESX.TextUI('Text')
ESX.HideUI()
```

```lua
-- ✅ New
lib.showTextUI('Text', { position = 'left-center' })
lib.hideTextUI()
```

**`ESX.Progressbar` to `lib.progressBar`**

```lua
-- ❌ Old
ESX.Progressbar('repair', 'Repairing...', 5000, { FreezePlayer = false, animation = { ... } }, {}, {}, {}, function() end, function() end)
```

```lua
-- ✅ New
if lib.progressBar({ duration = 5000, label = 'Repairing...', anim = { dict = 'mini@repair', clip = 'fixing_a_player' }, disable = { car = true, combat = true } }) then
  -- completed
else
  -- cancelled
end
```

**`DrawMarker` plus distance loop to `ox_target`**

```lua
-- ❌ Old
CreateThread(function()
  while true do
    local sleep = 1000
    local ped = PlayerPedId()
    local coords = GetEntityCoords(ped)
    if #(coords - Config.Shop.Coords) < 20.0 then
      sleep = 0
      DrawMarker(1, Config.Shop.Coords.x, Config.Shop.Coords.y, Config.Shop.Coords.z - 1.0, 0,0,0,0,0,0, 1.5,1.5,0.8, 0,150,255,120, false, true, 2, false, nil, nil, false)
      if #(coords - Config.Shop.Coords) < 2.0 then
        ESX.ShowHelpNotification('Press ~INPUT_CONTEXT~')
        if IsControlJustPressed(0, 38) then openShop() end
      end
    end
    Wait(sleep)
  end
end)
```

```lua
-- ✅ New
exports.ox_target:addBoxZone({
  coords = Config.Shop.Coords,
  size   = vec3(1.5, 1.5, 2.0),
  options = { { name = 'shop:open', label = 'Open shop', icon = 'fa-solid fa-shop', onSelect = openShop } },
  distance = 2.0,
})
```

**ESX inventory methods to ox_inventory**

```lua
-- ❌ Old
xPlayer.addInventoryItem('bread', 1)
xPlayer.removeInventoryItem('bread', 1)
xPlayer.getInventoryItem('bread')
```

```lua
-- ✅ New
exports.ox_inventory:AddItem(source, 'bread', 1)
exports.ox_inventory:RemoveItem(source, 'bread', 1)
exports.ox_inventory:GetItemCount(source, 'bread', nil, false)
```

---

## 22. Code Review Checklist, ox-powered

When reviewing ESX code, check:

Does `fxmanifest.lua` include `lua54 'yes'`? Does it use `@es_extended/imports.lua`? Does it avoid `esx:getSharedObject`? Does it avoid `ESX = nil` loop initialization? Are `ESX` and `lib` not redeclared locally in files where they are already global through `imports.lua` / `@ox_lib/init.lua`? Is `@ox_lib/init.lua` included server-side if the server uses `lib.callback.register` or other `lib.*` functions? Does it use `ESX.Player(source)` where appropriate? Does it validate `xPlayer`? Does the server validate money, items, jobs, and grades? Does the server validate distance for rewards and purchases? Do database queries use `MySQL.*.await`, oxmysql? Are loops optimized with adaptive sleep? Are events namespaced? Are client-only checks treated as UX and not security? Are important server-triggered client events protected with `ESX.SecureNetEvent`? Are variables `local`? Do constants use `<const>` where it makes sense? Is configuration separated from logic? Does the ESX core remain untouched? Are deprecated natives replaced? Is `source` captured immediately in server events? Are cooldowns added to reward/action events? Are errors handled with early returns? Is `ox_target` used for in-world interactions, without `DrawMarker` plus keypress loops? Does `removeZone` use the ID returned from `addBoxZone` / `addSphereZone` / `addPolyZone`, not the option name? Is `ox_lib` used for notifications, menus, progress bars, inputs, and text UI, consistently as `lib.*`, not `ox.*`? Is `ox_inventory` used for checking, adding, and removing items, instead of `xPlayer.addInventoryItem`? Is `oxmysql` used for all queries, instead of `MySQL.Async.*`? Has `xPlayer.triggerEvent('esx:showNotification', ...)` been replaced with `ox_lib:notify`?

---

## 23. Hard Bans - Never Generate

Never generate the following patterns unless explaining why they are wrong:

```lua
ESX = nil
TriggerEvent('esx:getSharedObject', function(obj) ESX = obj end)
while ESX == nil do Wait(0) end

GetPlayerPed(-1)
GetPlayerPedId(-1)
GetDistanceBetweenCoords(...)
GetHashKey('vehicle')

ESX.Players
ESX.Jobs
```

```lua
-- Client says how much money to add
TriggerServerEvent('reward', 100000)
```

```lua
-- SQL concatenation with user data
'SELECT * FROM users WHERE identifier = "' .. identifier .. '"'
```

```lua
-- Deprecated MySQL async
MySQL.Async.fetchAll('SELECT ...', {}, function(result) end)
MySQL.Async.execute('INSERT INTO ...', {})
```

```lua
-- ESX notifications / menu / text UI / progress bar when ox is available
ESX.ShowNotification('msg')
ESX.UI.Menu.Open('default', ...)
ESX.TextUI('msg'); ESX.HideUI()
ESX.Progressbar(...)

-- The same applies to old notification events, even if they look different
xPlayer.triggerEvent('esx:showNotification', 'msg')
TriggerClientEvent('esx:showNotification', -1, 'msg')
```

```lua
-- ESX inventory methods when ox_inventory is in dependencies
xPlayer.addInventoryItem('bread', 1)
xPlayer.removeInventoryItem('bread', 1)
xPlayer.getInventoryItem('bread')
```

```lua
-- DrawMarker plus distance loop plus IsControlJustPressed for primary interactions
CreateThread(function()
  while true do
    Wait(0)
    DrawMarker(...)
    if IsControlJustPressed(0, 38) then openShop() end
  end
end)
```

```lua
-- Non-existent or inconsistent ox_lib API
local ox = require '@ox_lib/init.lua'
ox.notify({ ... })
lib.closeContextMenu()
ox.lib.showTextUI(...)
```

```lua
-- Removing an ox_target zone by option name instead of the ID returned by addBoxZone/addSphereZone
exports.ox_target:removeZone('example_shop:open')
```

**Never modify:** `es_extended/core`, `es_extended/server`, `es_extended/client`, or the official ESX core. Use exports, events, callbacks, configuration, and separate resources.

---

## 24. Response Style for This Skill

**When the user asks for a script:**

* provide complete files,
* in `fxmanifest.lua`, include `ox_lib`, `ox_inventory`, `ox_target`, and `oxmysql` in dependencies, and include `@ox_lib/init.lua` in `server_scripts` if you use `lib.callback` server-side,
* use modern ESX initialization, without unnecessary `local ESX` / `local ox = require ...` redeclarations if `imports.lua` / `@ox_lib/init.lua` are already in the manifest,
* use `ox_target` for in-world interactions,
* use `lib.*` for UI, notifications, progress, text UI, inputs, and callbacks,
* use `ox_inventory` for item operations,
* use `oxmysql` for database work,
* use ox_lib callbacks by default, `lib.callback`; use `ESX.RegisterServerCallback` only for backward compatibility if the user requests it,
* provide paste-ready code,
* add short installation steps,
* mention SQL only when needed,
* briefly explain security-relevant parts.

**When the user asks to fix a script:**

* identify the bug,
* replace deprecated ESX patterns with ox equivalents,
* replace `ESX.UI.Menu` / `ESX.ShowNotification` / `ESX.TextUI` / `ESX.Progressbar` / `xPlayer.triggerEvent('esx:showNotification', ...)` with `lib.*` / `ox_lib:notify`,
* replace `xPlayer.addInventoryItem` and similar methods with `ox_inventory` calls,
* replace `MySQL.Async.*` with `MySQL.*.await`,
* replace `DrawMarker` plus interaction loops with `ox_target` zones, with correct removal by ID,
* return corrected code,
* explain what changed,
* warn about security issues if present.

**When the user asks for optimization:**

* remove unnecessary `Wait(0)` loops,
* add adaptive sleep,
* safely cache ped/player data,
* use events instead of polling where possible,
* move important validation server-side,
* move in-world interactions to `ox_target`,
* use ox_lib callbacks and context menus instead of `ESX.UI.Menu`,
* use `ox_inventory` exports instead of `xPlayer` methods for items.

**When the user asks for “latest ESX”:**

* use `@es_extended/imports.lua` in `fxmanifest.lua`, do not use `esx:getSharedObject`,
* use `ESX.Player` and `ESX.ExtendedPlayers`,
* use `ESX.SecureNetEvent` for server-only client events,
* use `ox_target`, `ox_lib`, `ox_inventory`, and `oxmysql` for all UI, inventory, and database work,
* use Lua 5.4,
* do not modify the ESX core.

---

## 25. Gold Standard Mini Template, ox-powered

**fxmanifest.lua**

```lua
fx_version 'cerulean'
game 'gta5'

description 'Modern ESX Legacy resource (ox-powered)'
author 'YourName'
version '1.0.0'

lua54 'yes'

shared_scripts {
  '@es_extended/imports.lua',
  'config.lua',
}

client_scripts {
  '@ox_lib/init.lua',
  'client.lua',
}

server_scripts {
  '@oxmysql/lib/MySQL.lua',
  '@ox_lib/init.lua',
  'server.lua',
}

dependencies {
  'es_extended',
  'oxmysql',
  'ox_lib',
  'ox_inventory',
  'ox_target',
}
```

**config.lua**

```lua
Config = {}

Config.Debug    = false
Config.Locale   = 'en'
Config.Distance = 2.0
```

**client.lua**

```lua
-- ESX and lib are global because of '@es_extended/imports.lua' and '@ox_lib/init.lua'

local isPlayerLoaded = false
local playerData     = {}
local ped            = PlayerPedId()

CreateThread(function()
  while not ESX.IsPlayerLoaded() do
    Wait(250)
  end

  isPlayerLoaded = true
  playerData     = ESX.GetPlayerData()
  ped            = PlayerPedId()
end)

AddEventHandler('esx:playerLoaded', function(xPlayer)
  isPlayerLoaded = true
  playerData     = xPlayer
  ped            = PlayerPedId()
end)

AddEventHandler('esx:setJob', function(job)
  playerData.job = job
end)

AddEventHandler('esx:playerPedChanged', function(newPed)
  ped = newPed
end)
```

**server.lua**

```lua
-- ESX is global because of '@es_extended/imports.lua'

local RESOURCE_NAME <const> = GetCurrentResourceName()

AddEventHandler('onResourceStart', function(resourceName)
  if resourceName ~= RESOURCE_NAME then return end
  print(('[%s] loaded.'):format(RESOURCE_NAME))
end)
```

---

## 26. ox_inventory Quick Reference

```lua
-- server
exports.ox_inventory:RegisterUse('item', function(source) end)
exports.ox_inventory:AddItem(source, 'item', count, metadata, slot)
exports.ox_inventory:RemoveItem(source, 'item', count, metadata, slot)
exports.ox_inventory:GetItemCount(source, 'item', metadata, strict)
exports.ox_inventory:CanAddItem(source, 'item', count, metadata)
exports.ox_inventory:CanRemoveItem(source, 'item', count, metadata)
exports.ox_inventory:SetMetadata(source, 'item', metadata, slot)
exports.ox_inventory:ClearInventory(source, filter)
exports.ox_inventory:ConfiscateInventory(source)

-- client, UI/UX only
exports.ox_inventory:openInventory(invType, invId)
exports.ox_inventory:closeInventory()
exports.ox_inventory:GetItemCount(item, metadata, strict)
exports.ox_inventory:Search(invType, search)
```

---

## 27. ox_target Quick Reference

```lua
-- Single zones. Each function returns a zone ID.
local zoneId = exports.ox_target:addBoxZone({ coords, size, rotation, options, distance, debug })
local zoneId = exports.ox_target:addSphereZone({ coords, radius, options, distance, debug })
local zoneId = exports.ox_target:addPolyZone({ points, thickness, options, distance, debug })

-- Entities
exports.ox_target:addLocalEntity(ped, options, distance)
exports.ox_target:addGlobalVehicle(options, distance)
exports.ox_target:addGlobalPeds(options, distance)
exports.ox_target:addGlobalObject(options, distance)
exports.ox_target:addLocalObject(ent, options, distance)
exports.ox_target:addLocalVehicle(ent, options, distance)

-- Management
exports.ox_target:removeZone(zoneId)              -- ID from addBoxZone/addSphereZone/addPolyZone
exports.ox_target:removeGlobalVehicle('name')     -- option name
exports.ox_target:removeLocalEntity(ped, 'name')  -- option name
exports.ox_target:disableTargeting(playerId, flag, toggle)
```

---

## 28. ox_lib Quick Reference

```lua
-- callbacks
lib.callback.register('name', function(source, ...) return ... end)
lib.callback('name', async, function(result) end, ...)
local result = lib.callback.await('name', source, ...)

-- notifications
lib.notify({ title, description, type, position, duration, icon })

-- progress
local ok = lib.progressBar({ duration, label, useWhileDead, canCancel, disable, anim, prop })
local ok = lib.progressCircle({ duration, label, useWhileDead, canCancel, disable, anim, prop })

-- text UI
lib.showTextUI(message, options)
lib.hideTextUI()
lib.isTextUIOpen()

-- input
local input = lib.inputDialog(heading, fields)

-- menu / context
lib.registerMenu({ id, title, position, options }, onSelected, onClose)
lib.showMenu('id'); lib.hideMenu('id')

lib.registerContext({ id, title, options })
lib.showContext('id'); lib.hideContext()

-- zone helpers, visual use only, not for primary interactions
lib.zones.box({ coords, size, rotation, debug, inside, onEnter, onExit })
lib.zones.sphere({ coords, radius, inside, onEnter, onExit })
lib.zones.poly({ points, thickness, inside, onEnter, onExit })
```

---

## 29. Final Quality Bar

A script generated by this skill should be:

compatible with modern ESX Legacy, based on the ox ecosystem (`ox_lib`, `ox_inventory`, `ox_target`, `oxmysql`), secure by default, cleanly organized, easy to configure, optimized enough for production, readable for junior developers, strict about server-side validation, free from deprecated ESX initialization, free from `MySQL.Async.*`, free from `ESX.UI.Menu` / `ESX.ShowNotification` / `ESX.TextUI` / `ESX.Progressbar` / `xPlayer.triggerEvent('esx:showNotification', ...)` when ox is available, free from `xPlayer.addInventoryItem` / `xPlayer.removeInventoryItem` when `ox_inventory` is available, free from `DrawMarker` plus keypress loops when `ox_target` is available, free from unnecessary global variables and unnecessary `ESX` / `lib` redeclarations, and ready to paste into a FiveM resource.

If confidence about a specific ESX API is low, check the official ESX Legacy / ox_lib / ox_inventory / ox_target documentation before answering.
