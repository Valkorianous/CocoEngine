# CocoUI

A **modular** system for automatically discovering and connecting **UI behavior** based on Roblox’s `CollectionService` tags. Simply place your UI definition modules in a specified folder, and the framework will:

1. **Recursively load** each module (or array of modules).  
2. **Listen** for CollectionService tags (e.g. `"PrintButton"`, `"InventoryButton"`) declared in those modules.  
3. **Auto-connect** event callbacks (e.g. `OnClick`, `OnMouseEnter`, `OnChanged`) for any UI objects with matching tags.  
4. **Detach** event connections when tags are removed or instances are destroyed.

---

## Table of Contents

1. [Overview](#overview)  
2. [Installation](#installation)  
3. [Key Concepts](#key-concepts)  
4. [API Reference](#api-reference)  
   - [UIFramework.Init(modulesFolder)](#uiframeworkinitmodulesfolder)  
   - [UIFramework:loadModulesRecursively(folder)](#uiframeworkloadmodulesrecursivelyfolder)  
   - [UIFramework:registerModule(definition)](#uiframeworkregistermoduledefinition)  
   - [UIFramework:connectTag(tagName-definition)](#uiframeworkconnecttagtagname-definition)  
   - [UIFramework:attachInstance(definition-tagName-instance)](#uiframeworkattachinstancedefinition-tagname-instance)  
   - [UIFramework:detachInstance(definition-tagName-instance)](#uiframeworkdetachinstancedefinition-tagname-instance)  
5. [Supported Events](#supported-events)  
6. [Example Usage](#example-usage)  
7. [Optional: UIMessageBus](#optional-uimessagebus)  
8. [Notes & Best Practices](#notes--best-practices)

---

## Overview

**UIFramework** leverages Roblox’s `CollectionService` to automatically watch for UI elements that have particular tags. Whenever it detects an instance with a tag that a **definition module** cares about, it automatically wires up relevant event callbacks (like `OnClick`, `OnChanged`, etc.) and calls optional lifecycle hooks (`OnInstanceInit`, `OnInstanceRemoved`).

---

## Installation

1. **Place** `UIFramework.lua` in a known location, for example:

    - `ReplicatedStorage/UIFramework/UIFramework.lua`
2. **Create a Folder** (like `ReplicatedStorage/UIFramework/Modules`) to hold your ModuleScripts that define UI behavior.  
3. **Require and Initialize** the framework in a script (commonly a **LocalScript** in `StarterPlayerScripts` if these are client-side UIs):

    ```lua
    local ReplicatedStorage = game:GetService("ReplicatedStorage")
    local UIFrameworkFolder = ReplicatedStorage:WaitForChild("UIFramework")
    local UIFramework = require(UIFrameworkFolder:WaitForChild("UIFramework"))

    -- The folder containing your UI definition modules:
    local modulesFolder = UIFrameworkFolder:WaitForChild("Modules")

    -- Initialize the UIFramework
    local frameworkInstance = UIFramework.Init(modulesFolder)
    ```

Once called, the framework will recursively require all ModuleScripts in `modulesFolder` and wire up any tags those definitions specify.

---

## Key Concepts

### 1. Definition Tables

Each ModuleScript in your `Modules` folder can return either:
- A **single definition table**, or
- An **array** of definition tables.

A definition table has fields such as:

    {
        Tags = { "ButtonTag" },         -- List of tags to watch for
        OnClick = function(instance)    -- Called if instance is a TextButton / ImageButton
            print("Button clicked:", instance.Name)
        end,
        OnInstanceInit = function(instance)
            -- Called once when the instance is first connected
        end,
        OnInstanceRemoved = function(instance)
            -- Called when the instance is untagged or destroyed
        end,
        -- More callbacks: OnMouseEnter, OnMouseLeave, OnChanged, etc.
        Submodules = { 
            -- nested definitions if desired
        }
    }

### 2. Event Mapping

The framework automatically connects these definition keys:
- `OnClick` → `MouseButton1Click`
- `OnMouseEnter` → `MouseEnter`
- `OnMouseLeave` → `MouseLeave`
- `OnChanged` → `Changed`

You can add more in the source code (e.g., `OnInputBegan`).

### 3. Tag Discovery

It uses `CollectionService:GetInstanceAddedSignal(tagName)` and `CollectionService:GetInstanceRemovedSignal(tagName)` to detect when instances gain or lose a particular tag. When a match occurs, the framework “attaches” or “detaches” callbacks appropriately.

---

## API Reference

### UIFramework.Init(modulesFolder)

Initializes the UIFramework by loading definition modules from `modulesFolder`. Returns the new UIFramework instance.

- **Parameters**  
  `modulesFolder` (Instance) – A Folder containing your ModuleScripts.
- **Returns**  
  The created UIFramework instance (with its own `ActiveConnections`).

---

### UIFramework:loadModulesRecursively(folder)

Recursively finds ModuleScripts in `folder`. For each ModuleScript, it calls `require`, then registers the returned table(s).

- **Parameters**  
  `folder` (Instance)

---

### UIFramework:registerModule(definition)

Takes one definition table (or sub-definition in an array), reads `.Tags`, and calls `connectTag` for each. If `.Submodules` is present, registers them too.

- **Parameters**  
  `definition` (table conforming to `UIFrameworkDefinition`)

---

### UIFramework:connectTag(tagName, definition)

Watches for existing and future instances with `tagName`. For each instance, calls `attachInstance(definition, tagName, instance)`. If the instance loses the tag, `detachInstance` is called.

- **Parameters**  
  `tagName` (string), `definition` (UIFrameworkDefinition)

---

### UIFramework:attachInstance(definition, tagName, instance)

Connects events (e.g. `OnClick`, `OnMouseEnter`) and calls `OnInstanceInit` if present. Stores the event connections in an internal table so they can be cleaned up later.

- **Parameters**  
  `definition` (UIFrameworkDefinition), `tagName` (string), `instance` (Instance)

---

### UIFramework:detachInstance(definition, tagName, instance)

Disconnects the events attached by `attachInstance`. Calls `OnInstanceRemoved(instance)` if defined.

- **Parameters**  
  `definition` (UIFrameworkDefinition), `tagName` (string), `instance` (Instance)

---

## Supported Events

Within the `SupportedEvents` table of `UIFramework.lua`, the following keys are mapped:

- `OnClick` → `MouseButton1Click` (for `TextButton` or `ImageButton`)
- `OnMouseEnter` → `MouseEnter` (for any `GuiObject`)
- `OnMouseLeave` → `MouseLeave` (for any `GuiObject`)
- `OnChanged` → `Changed` (for any `GuiObject`)

You can **add more** if needed. The system checks if `typeof(callback) == "function"` and if the instance type supports that event.

---

## Example Usage

1. **Tag Your UI**: In Roblox Studio, select a `TextButton`, open Properties, and add `"PrintButton"` to **Tags**.
2. **Create a Definition Module** (`PrintModule.lua`) in `UIFramework/Modules`:

       return {
         Tags = { "PrintButton" },
         OnInstanceInit = function(instance)
             print("Found a PrintButton:", instance.Name)
         end,
         OnClick = function(instance)
             print("Clicked a PrintButton:", instance.Name)
         end
       }

3. **Initialize** in a script:

       local ReplicatedStorage = game:GetService("ReplicatedStorage")
       local UIFrameworkFolder = ReplicatedStorage:WaitForChild("UIFramework")
       local UIFramework = require(UIFrameworkFolder:WaitForChild("UIFramework"))
       
       local modulesFolder = UIFrameworkFolder:WaitForChild("Modules")
       local frameworkInstance = UIFramework.Init(modulesFolder)

When you press **Play**, any UI object tagged `"PrintButton"` gets `OnInstanceInit` called, and clicking it calls `OnClick`.

---

## Optional: UIMessageBus

You can include a simple pub/sub module (e.g. `UIMessageBus.lua`) for in-game communication among UI modules:

    local UIMessageBus = require(script.Parent.UIMessageBus)

    -- Publish
    UIMessageBus.Publish("SOME_TOPIC", { foo = "bar" })

    -- Subscribe
    local conn = UIMessageBus.Subscribe("SOME_TOPIC", function(data)
        print("Got data:", data.foo)
    end)

This keeps modules decoupled rather than directly requiring each other.

---

## Notes & Best Practices

- **No Hardcoding**: You never manually connect `MouseButton1Click` or `MouseEnter`; just declare `OnClick`, `OnMouseEnter` in your definition.
- **Array Returns**: A single module can return multiple definitions. The framework registers each one.
- **Submodules**: If your definition has `Submodules`, those are also registered automatically.
- **Cleanup**: `OnInstanceRemoved` and the auto-disconnect in `detachInstance` help prevent memory leaks or leftover event connections.
- **Performance**: This is usually fine even for large UIs. Just avoid hooking `OnChanged` for many objects if you don’t need it frequently.

Copy & paste this entire Markdown block into your README or documentation, and you should have a clean, fully formatted page.
