# Runtime Game Explorer

I made a runtime explorer for devs a couple years back to help debug live games.

It lets you view your entire game, see all properties, write properties, write code, etc. for your client , server view, or other clients

https://github.com/user-attachments/assets/54ab3e51-2a4c-4b3c-b57f-ceb3444f212a

- Large instance trees with virtual scrolling(opening a 35,000 instance folder only causes about ~0.5 seconds of lag)
- Simple context menu to cut, copy, paste, or delete instances
- Simple selection behavior(LeftAlt to select parts, otherwise selects models), with simple transform tools
- Real time output/dev console for reading the output of any client or the server , and writing code
- Real time updating of properties under the selected instance
- Writing properties
- Attributes support

# How to get

Download the .rbxm from the latest release, and drop it into studio

https://github.com/TylerAtStarboard/RuntimeGameExplorer/releases/tag/V4.2

---

# Hooks API

The hooks module is automatically moved to `ReplicatedStorage` at runtime. Require it from there:

```lua
local Hooks = require(game.ReplicatedStorage:WaitForChild("RuntimeExplorerHooks"))
```

Client methods live under `Hooks.Client`, server methods live under `Hooks.Server`. 

A couple example scripts are included in the `ClientDevScripts` folder.

---

## Client API

### Selection

#### `Hooks.Client:ObserveSelected(fn: (Instance?) -> (cleanup)) -> stop`

Observe the currently selected instance. Fires immediately with the current value, then again whenever it changes. Return a cleanup function from the callback — it runs when the selection changes or the observer is stopped.

```lua
local stop = Hooks.Client:ObserveSelected(function(instance: Instance?)
    print("Selected:", instance)
    return function()
        print("No longer selected:", instance)
    end
end)

stop() -- cleanup the observer when we no longer need it!
```

#### `Hooks.Client:GetSelected() -> Instance?`

Returns the currently selected instance, or `nil` if nothing is selected or the tool is closed. For one-off checks without setting up an observer.

```lua
local inst = Hooks.Client:GetSelected()
```

#### `Hooks.Client:SelectInstance(instance: Instance, openTool: boolean?)`

Programmatically select an instance in the local explorer, expanding its ancestors in the tree and scrolling to it. If `openTool` is true, the explorer tool will be force-opened first.

```lua
Hooks.Client:SelectInstance(workspace.SomePart)
Hooks.Client:SelectInstance(workspace.SomePart, true) -- also opens the tool
```

### Tool Control

#### `Hooks.Client:ObserveToolOpened(fn: (boolean) -> (cleanup)) -> stop`

Observe whether the explorer tool is open or closed. Fires immediately with the current state, then again whenever it changes.

```lua
local stop = Hooks.Client:ObserveToolOpened(function(isOpen: boolean)
    print("Tool is now", if isOpen then "open" else "closed")
    return function()
        print("Tool state changed away from", isOpen)
    end
end)

stop()
```

#### `Hooks.Client:IsToolOpen() -> boolean`

Returns whether the explorer tool is currently open. For one-off checks without setting up an observer.

```lua
if Hooks.Client:IsToolOpen() then
    -- ...
end
```

#### `Hooks.Client:OpenTool()`

Opens the explorer tool.

#### `Hooks.Client:CloseTool()`

Closes the explorer tool.

#### `Hooks.Client:ToggleTool()`

Toggles the explorer tool open/closed.

---

## Server API

For FullAccess admins using server mode.

#### `Hooks.Server:ObserveSelectedByAdmin(admin: Player, fn: (Instance?) -> (cleanup)) -> stop`

Observes the currently selected instance for a FullAccess admin while viewing the server explorer. Fires immediately with the current value, then again whenever it changes. The observer is automatically cleaned up when the admin leaves the game.

```lua
local stop = Hooks.Server:ObserveSelectedByAdmin(admin, function(instance: Instance?)
    print(admin.Name, "selected", instance)
    return function()
        print(admin.Name, "deselected", instance)
    end
end)

stop()
```

#### `Hooks.Server:SelectInstanceForAdmin(admin: Player, instance: Instance, openTool: boolean?)`

Programmatically select a server instance for an admin who is currently viewing the server explorer. Expands the instance's ancestors so it becomes visible, then scrolls their explorer to it. If `openTool` is true, the explorer tool will be force-opened on the admin's client first. Does nothing if the admin is not in server mode or the instance cannot be reached.

```lua
Hooks.Server:SelectInstanceForAdmin(admin, workspace.SomePart)
Hooks.Server:SelectInstanceForAdmin(admin, workspace.SomePart, true) -- also opens the tool
```

---

# Updates

## V4.1

- **Added hooks!** You can now programmatically open/toggle the tool with your own binds, respond to explorer instance selection, force explorer selection on an instance, and more
- This should make it easier to work with this tool and bind it to your games
- A couple example scripts are included
  
Some examples:

```lua
-- Client Hook Example for listening to a selected instance

local Hooks = require(game.ReplicatedStorage:WaitForChild("RuntimeExplorerHooks"))

local stop = Hooks.Client:ObserveSelected(function(instance: Instance?)
    print("Selected:", instance)
    return function()
        print("No longer selected:", instance)
    end
end)
```

```lua
-- Client Hook Example for toggling the tool

local Hooks = require(game.ReplicatedStorage:WaitForChild("RuntimeExplorerHooks"))

ContextActionService:BindAction("InGameExplorer_Toggle", function(_, inputState) 
    if inputState ~= Enum.UserInputState.Begin then
        return
    end
    
    Hooks.Client:ToggleTool()
    
    return Enum.ContextActionResult.Pass
end, false, Enum.KeyCode.F3)

-- observe if the tool is opened and print to output
local stop = Hooks.Client:ObserveToolOpened(function(isOpen: boolean)
    print("Tool is now", if isOpen then "open" else "closed")
    return function()
        print("Tool state changed away from", isOpen)
    end
end)
```

```lua
-- Client Hook Example, open the tool when you approach a plant

local ReplicatedStorage = game:GetService("ReplicatedStorage")
local RunService = game:GetService("RunService")
local CollectionService = game:GetService("CollectionService")
local Player = game:GetService("Players").LocalPlayer
local Hooks = require(ReplicatedStorage:WaitForChild("RuntimeExplorerHooks"))
local ObserveTag = require(ReplicatedStorage.SharedModules.Management.Observers.ObserveTag)

local Character = Player.Character or Player.CharacterAdded:Wait()

-- Open tool to the nearest plant if close enough
local function GetClosestPlant()
    local charPos = Character:GetPivot().Position
    local closest : Model = nil
    local lastDist = 8
    
    for i, plant : Model in CollectionService:GetTagged("Plant") do 
        local dist = vector.magnitude(plant:GetPivot().Position - charPos)
        if dist < lastDist then
            closest = plant
            lastDist = dist
        end
    end
    return closest
end

RunService.Heartbeat:Connect(function()
    local closest = GetClosestPlant()
    if closest then
        Hooks.Client:SelectInstance(closest, true)
    end
end)
```

Plant example:

https://github.com/user-attachments/assets/494a6ba9-603d-48c2-8a98-2fbdecadc7de


## V4.2

Fixed respawn issue with the dev scripts
