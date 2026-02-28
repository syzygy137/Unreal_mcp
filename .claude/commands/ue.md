# Unreal MCP Skill — Tool Usage Guide & Patterns

You are controlling Unreal Engine via the McpAutomationBridge MCP server. This skill contains verified patterns, known bugs, and workflow rules. Follow them strictly.

---

## WORKFLOW RULES

### Stop-and-Flag Protocol
When you encounter a tool limitation (timeout, missing parameter, unsupported property):
1. **Try at most 2 alternative approaches**
2. **If both fail, STOP and tell the user**: "This tool can't do X. We could fix it by modifying [C++ plugin / TS schema / etc]. Want me to do that, or should we work around it?"
3. **Never burn more than 3 attempts** on the same operation with different parameter guesses

### Before Writing Blueprint Nodes
- Always `get_graph_details` first to see existing nodes and IDs
- Always `get_node_details` or `get_pin_details` before connecting pins — never guess pin names
- After wiring, always `compile` and check for errors

### Save Protocol
- MCP-created/modified assets are **in-memory only** until saved
- After completing a set of changes, call `control_editor` → `save_all`
- IMC_Default mouse/scroll bindings are especially prone to being lost on crash — always save after adding them
- Tell the user to Ctrl+Shift+S after testing

---

## TOOL LOADING

Tools are deferred. Before using any MCP tool, load it first:
```
ToolSearch: select:mcp__ue__manage_blueprint
ToolSearch: select:mcp__ue__manage_input
ToolSearch: select:mcp__ue__animation_physics
ToolSearch: select:mcp__ue__control_editor
ToolSearch: select:mcp__ue__manage_asset
ToolSearch: select:mcp__ue__inspect
ToolSearch: select:mcp__ue__control_actor
ToolSearch: select:mcp__ue__manage_tools
```

Use `manage_tools` → `get_status` as a quick connection test.

---

## TOOL REFERENCE — VERIFIED PATTERNS

### manage_blueprint

#### Create Nodes
```json
{"action": "create_node", "blueprintPath": "/Game/Path/BP", "graphName": "EventGraph",
 "nodeType": "K2Node_CallFunction", "memberName": "FunctionName", "memberClass": "ClassName"}
```

**EnhancedInputAction nodes** require the `inputAction` parameter (we added this to the schema):
```json
{"action": "create_node", "blueprintPath": "/Game/Path/BP", "graphName": "EventGraph",
 "nodeType": "K2Node_EnhancedInputAction", "inputAction": "IA_ActionName"}
```

**Common node types:**
- `K2Node_CallFunction` — function calls (memberName + memberClass)
- `K2Node_EnhancedInputAction` — Enhanced Input events (requires inputAction param)
- `K2Node_VariableGet` / `K2Node_VariableSet` — variable access
- `K2Node_DynamicCast` — cast nodes
- `K2Node_IfThenElse` — branch

**AnimGraph node types (for AnimBlueprints):**
- `AnimGraphNode_SequencePlayer` — looping animation playback
- `AnimGraphNode_SequenceEvaluator` — single frame evaluation (e.g., idle pose at frame 0)
- `AnimGraphNode_BlendListByBool` — blend between two poses by boolean
- `AnimGraphNode_Slot` — montage slot (DefaultSlot)
- `AnimGraphNode_ModifyBone` — bone transform modification
- `AnimGraphNode_Root` — output pose (exists by default)
- `AnimGraphNode_TwoWayBlend` — blend by float alpha

#### Connect Pins
```json
{"action": "connect_pins", "blueprintPath": "/Game/Path/BP",
 "fromNodeId": "GUID", "fromPinName": "PinName",
 "toNodeId": "GUID", "toPinName": "PinName"}
```

#### Set Pin Default Values
```json
{"action": "set_pin_default_value", "blueprintPath": "/Game/Path/BP",
 "nodeId": "GUID", "pinName": "PinName", "value": "TheValue"}
```

For object references (like AnimMontage), use the full asset path:
```json
{"pinName": "AnimMontage", "value": "/Game/Path/MyMontage"}
```

#### Set Node Properties (AnimGraph)
```json
{"action": "set_node_property", "blueprintPath": "/Game/Path/ABP",
 "nodeId": "GUID", "propertyName": "Sequence",
 "value": "/Game/Path/AnimSequence"}
```
Works for: SequencePlayer.Sequence, SequenceEvaluator.Sequence, ModifyBone.BoneToModify, ModifyBone.TranslationMode/RotationMode/ScaleMode

#### Add Variables
```json
{"action": "add_variable", "blueprintPath": "/Game/Path/BP",
 "variableName": "MyVar", "variableType": "Boolean"}
```

#### Compile
```json
{"action": "compile", "blueprintPath": "/Game/Path/BP"}
```

### manage_input

#### Create Input Action
```json
{"action": "create_input_action", "name": "IA_Name", "path": "/Game/Input"}
```

#### Add Mapping to Context
```json
{"action": "add_mapping", "contextPath": "/Game/Input/IMC_Default",
 "actionPath": "/Game/Input/IA_Name", "key": "KeyName"}
```

**Common key names:** `W`, `A`, `S`, `D`, `SpaceBar`, `LeftShift`, `LeftControl`, `MouseX`, `MouseY`, `MouseWheelAxis`, `LeftMouseButton`, `RightMouseButton`, `E`, `Q`, `F`

#### Get Info
```json
{"action": "get_input_info", "assetPath": "/Game/Input/IMC_Default"}
```
Note: uses `assetPath`, NOT `contextPath` for get_input_info.

### animation_physics

#### Set Blend Out on Montage
```json
{"action": "set_blend_out", "assetPath": "/Game/Path/Montage", "blendTime": 0.35}
```
**WARNING:** This frequently times out. If it does, tell the user to set it manually in the montage editor (Details panel → Blend Out → Blend Time).

#### Set Blend In on Montage
```json
{"action": "set_blend_in", "assetPath": "/Game/Path/Montage", "blendTime": 0.25}
```

#### Create Animation Blueprint
```json
{"action": "create_anim_blueprint", "name": "ABP_Name", "savePath": "/Game/Path",
 "skeletonPath": "/Game/Path/Skeleton"}
```
Use `create_anim_blueprint` — most reliable variant.

#### Create Montage — UNRELIABLE
`create_montage` consistently times out. Workaround: check if a montage already exists in the project (search assets), or tell the user to create it manually (right-click AnimSequence → Create → AnimMontage).

### control_editor

#### Save All
```json
{"action": "save_all"}
```

#### Console Command
```json
{"action": "console_command", "command": "the command"}
```

#### Play/Stop PIE
```json
{"action": "play"}
{"action": "stop"}
```
**CAUTION:** Modifying Blueprints during Play-In-Editor can crash the editor. Always stop PIE first.

### inspect

#### Inspect Object
```json
{"action": "inspect_object", "objectPath": "/Game/Path/Asset.AssetName"}
```
Note: needs `Package.ObjectName` format for assets.

#### Get/Set Property
```json
{"action": "get_property", "objectPath": "/Game/Path/Asset.Asset", "propertyName": "PropName"}
{"action": "set_property", "objectPath": "/Game/Path/Asset.Asset", "propertyName": "PropName", "value": "val"}
```
**LIMITATION:** Struct properties (like BlendOut.BlendTime on montages) are NOT accessible. AnimSequence properties (root motion settings) are NOT modifiable via MCP.

### manage_asset

#### Search Assets
```json
{"action": "search_assets", "searchText": "keyword", "classNames": ["AnimMontage"]}
```

#### Save Specific Asset
Not directly supported. Use `control_editor` → `save_all` or `console_command` → `obj savepackage`.

### control_actor

#### Set Blueprint Variables at Runtime
```json
{"action": "set_blueprint_variables", "actorName": "BP_Hero_C_0",
 "variables": {"MaxWalkSpeed": 120}}
```

---

## KNOWN BUGS & WORKAROUNDS

### add_variable Always Creates Integer
**Bug:** `manage_blueprint` → `add_variable` with `variableType: "Float"` (or "Double", "Real") always creates an integer variable. The `variablePinType` parameter is also ignored.
**Workaround:** Use Boolean variables with BlendByBool instead of Float with TwoWayBlend. Or use integer values where the range works (e.g., raw velocity 0-120 instead of normalized 0.0-1.0).

### create_montage Timeout
**Bug:** `animation_physics` → `create_montage` consistently times out after 30s.
**Workaround:** Search for existing montages in the project. Or tell user to create manually: right-click AnimSequence in Content Browser → Create → AnimMontage.

### set_blend_in / set_blend_out Timeout
**Bug:** These actions on animation_physics frequently time out.
**Workaround:** Tell user to set blend times manually in the montage editor.

### IMC_Default Bindings Lost on Crash/Restart
**Bug:** Mouse (MouseX, MouseY) and scroll (MouseWheelAxis) bindings on IMC_Default don't persist across editor crashes. The WASD/Space bindings do persist (they're in DefaultGame.ini).
**Workaround:** Re-add them via MCP after every restart, then `save_all` and tell user to Ctrl+Shift+S.

### ModifyBone Z-Height Sink
**Bug:** Using ModifyBone with Replace Existing → (0,0,0) in Component Space zeros out the Z height too, causing the character to sink into the ground. Hips reference pose is at ~Z=76.6.
**Workaround:** Remove ModifyBone from the main AnimGraph pipeline. Only use it if you can selectively zero X/Y while preserving Z (not possible with a single ModifyBone node). PlayAnimMontage through a Slot node is the better approach for action animations.

### UE5 Uses Doubles, Not Floats
**Bug:** Math nodes like `Divide_FloatFloat` don't exist in UE5. They've been renamed to use Double.
**Fix:** Use `Divide_DoubleDouble`, `Greater_DoubleDouble`, `Multiply_DoubleDouble`, etc.

---

## UE5 BLUEPRINT CONVENTIONS

### Pin Names
- Exec pins: `execute` (input), `then` (output)
- EnhancedInputAction pins: `Started`, `Triggered`, `Ongoing`, `Canceled`, `Completed`, `ActionValue`, `ElapsedSeconds`
- Function return: `ReturnValue`
- Self reference: `self`
- AnimGraph poses: `Pose` (output), `BlendPose_0`, `BlendPose_1` (for blend nodes)
- BlendByBool: `bActiveValue` (the boolean input)

### BlendByBool Pose Convention (COUNTERINTUITIVE)
- `BlendPose_0` = the pose when bool is **TRUE**
- `BlendPose_1` = the pose when bool is **FALSE**
- `BlendTime_0` and `BlendTime_1` control transition duration for each direction

### Common UE5 Function Names
These are the CORRECT names that work with `create_node`:
- `GetMovementComponent` (NOT GetCharacterMovement — that's C++ only)
- `PlayAnimMontage` (memberClass: Character)
- `SetDoublePropertyByName` (memberClass: KismetSystemLibrary) — for setting any numeric property at runtime
- `GetVelocity` — returns FVector
- `VectorLength` — returns double
- `TryGetPawnOwner` — in AnimBP EventGraph
- `AddControllerYawInput` / `AddControllerPitchInput`
- `AddMovementInput`
- `GetControlRotation` / `GetForwardVector` / `GetRightVector`
- `Jump` (memberClass: Character)

### AnimBP Architecture Pattern
Standard setup for character with walk/idle/montage support:
```
EventGraph:
  Event Blueprint Update Animation → TryGetPawnOwner → GetVelocity → VectorLength
  → Greater_DoubleDouble (threshold: 10) → Set IsMoving (bool)

AnimGraph:
  SequencePlayer (walk loop) → BlendPose_0 (TRUE = moving)
  SequenceEvaluator (frame 0 idle) → BlendPose_1 (FALSE = stopped)
  Get IsMoving → BlendByBool.bActiveValue
  BlendByBool → Slot (DefaultSlot) → Output Pose
```
BlendTime_0 = 0.3s (stopping blend), BlendTime_1 = 0.2s (starting blend).
Montages play through the Slot node via PlayAnimMontage on the character.

---

## SCHEMA GAP PROTOCOL

When you discover a tool parameter that the C++ plugin reads but the TypeScript schema doesn't expose:

1. **Identify the gap**: Check `src/tools/consolidated-tool-definitions.ts` for the tool's properties
2. **Check the C++ handler**: Look in `plugins/McpAutomationBridge/Source/McpAutomationBridge/Private/` for what field name the C++ code reads from the payload
3. **Add the missing param**: Edit `consolidated-tool-definitions.ts`, add the property using `commonSchemas.stringProp` / `numberProp` / etc.
4. **Rebuild**: `cd Unreal_mcp && npm run build`
5. **The MCP server auto-restarts** with the new schema

If the C++ plugin itself doesn't support what you need:
1. **Tell the user** what's missing and where the fix would go
2. **Edit the C++ file** in `plugins/McpAutomationBridge/Source/McpAutomationBridge/Private/`
3. **Copy the modified file to the UE project**:
   ```bash
   cp plugins/McpAutomationBridge/Source/McpAutomationBridge/Private/<modified_file>.cpp \
      <UE_PROJECT_PATH>/Plugins/McpAutomationBridge/Source/McpAutomationBridge/Private/
   ```
4. **User must do a clean rebuild** — this is REQUIRED or UE will use the old cached binary:
   - Close Unreal Editor completely
   - Delete the plugin's compiled output in the UE project:
     ```
     <UE_PROJECT_PATH>/Plugins/McpAutomationBridge/Binaries/
     <UE_PROJECT_PATH>/Plugins/McpAutomationBridge/Intermediate/
     ```
   - Reopen the UE project — it will recompile the plugin from source automatically
5. **Verify** by testing the new functionality after UE finishes loading

Key source files:
- `McpAutomationBridge_BlueprintGraphHandlers.cpp` — node creation, pin connections, node properties
- `McpAutomationBridge_AnimationHandlers.cpp` — animation operations
- `McpAutomationBridge_AnimationAuthoringHandlers.cpp` — montage/blend operations
- `McpAutomationBridge_InputHandlers.cpp` — input action/mapping operations
- `src/tools/consolidated-tool-definitions.ts` — TypeScript tool schemas
- `src/tools/handlers/blueprint-handlers.ts` — TypeScript handler routing for blueprint ops
- `src/tools/handlers/animation-authoring-handlers.ts` — TypeScript handler routing for animation ops

---

## MESHY AI SKELETON NOTES

Meshy AI animations have NO separate root bone — Hips IS the root (bone index 0). This causes:
- Roll/dodge animations bake ALL movement into Hips (translation + rotation together)
- Force Root Lock kills the roll visual because roll rotation is ON the Hips
- Enable Root Motion extracts both translation and rotation, both lost without processing
- ModifyBone Replace (0,0,0) sinks the character because it zeros Z height too

Best approach for action animations: Use PlayAnimMontage through a Slot node in the AnimBP. Accept some visual displacement during montages, or tell user to edit the FBX in Blender to zero Hips XY translation keys while keeping rotation.

---

## CONNECTION SETUP

### WSL2 (Linux CLI → Windows UE)
```
UE Plugin: listens on 0.0.0.0:8090
MCP Server: connects to <Windows gateway IP>:8090
```
- Set `bAllowNonLoopback=True` in DefaultGame.ini under `[/Script/McpAutomationBridge.McpAutomationBridgeSettings]`
- Set `MCP_AUTOMATION_ALLOW_NON_LOOPBACK=true` in .mcp.json env
- Find Windows gateway IP: `ip route show default | awk '{print $3}'`

### Same Machine (localhost)
```
UE Plugin: listens on 127.0.0.1:8090
MCP Server: connects to 127.0.0.1:8090
```

### .mcp.json Template
```json
{
  "mcpServers": {
    "ue": {
      "command": "node",
      "args": ["<path-to-Unreal_mcp>/dist/cli.js"],
      "env": {
        "UE_HOST": "<host>",
        "UE_PORT": "8090",
        "MCP_AUTOMATION_ALLOW_NON_LOOPBACK": "true"
      }
    }
  }
}
```
