### `getModPaused() -> boolean`

Returns whether the mod is paused.

### `getActivityLevel() -> number`

Returns the current activity level.

### `getActivityLevelSmoothed() -> number`

Returns the current activity level smoothed over time.

### `getVariable(name: string) -> number`

Returns the current value of a variable defined in the mod JSON.

### `setVariable(name: string, value: number)`

Sets the current value of a variable defined in the mod JSON.
To avoid conflicts, prefer this to directly writing to the variable's address if the mod has any passive effects.

### `addToVariable(name: string, value: number)`

Adds a value to the current value of a variable defined in the mod JSON.
Accumulates portion rounded off when casting to the variable's type.

### `getDefinition(name: string) -> number`

Lookup a definition by name from the mod.
Definitions are named addresses that can be referenced in pointers.
Currently, definitions are only created when using a register copy hook in the mod JSON.

### `setDefinition(name: string, address: number)`

Set a definition by name in the mod.
Definitions are named addresses that can be referenced in pointers.
Currently, definitions are only created when using a register copy hook in the mod JSON.

### `isEffectActive(name: string) -> boolean`

Returns whether an effect is currently active.

### `readUint8(address: number) -> number`

Reads an unsigned byte from the specified address.

### `readUint16(address: number) -> number`

Reads a 16-bit unsigned integer from the specified address.

### `readUint32(address: number) -> number`

Reads a 32-bit unsigned integer from the specified address.

### `readUint64(address: number) -> number`

Reads a 64-bit unsigned integer from the specified address.

### `readInt8(address: number) -> number`

Reads a signed byte from the specified address.

### `readInt16(address: number) -> number`

Reads a 16-bit signed integer from the specified address.

### `readInt32(address: number) -> number`

Reads a 32-bit signed integer from the specified address.

### `readInt64(address: number) -> number`

Reads a 64-bit signed integer from the specified address.

### `readFloat(address: number) -> number`

Reads a 32-bit floating point number from the specified address.

### `readDouble(address: number) -> number`

Reads a 64-bit floating point number from the specified address.

### `writeUint8(address: number, value: number)`

Writes an unsigned byte to the specified address.

### `writeUint16(address: number, value: number)`

Writes a 16-bit unsigned integer to the specified address.

### `writeUint32(address: number, value: number)`

Writes a 32-bit unsigned integer to the specified address.

### `writeUint64(address: number, value: number)`

Writes a 64-bit unsigned integer to the specified address.

### `writeInt8(address: number, value: number)`

Writes a signed byte to the specified address.

### `writeInt16(address: number, value: number)`

Writes a 16-bit signed integer to the specified address.

### `writeInt32(address: number, value: number)`

Writes a 32-bit signed integer to the specified address.

### `writeInt64(address: number, value: number)`

Writes a 64-bit signed integer to the specified address.

### `writeFloat(address: number, value: number)`

Writes a 32-bit floating point number to the specified address.

### `writeDouble(address: number, value: number)`

Writes a 64-bit floating point number to the specified address.

### `writeBytes(address: number, bytes: table)`

Writes a sequence of bytes to the specified address.

### `readBytes(address: number, length: number) -> table`

Reads a sequence of bytes from the specified address.

### `memset(address: number, value: number, size: number)`

Sets a sequence of bytes to the specified value.

### `allocate(size: number, flProtect: number) -> number`

Allocates a block of memory with the specified size and [protection flags](https://learn.microsoft.com/en-us/windows/win32/Memory/memory-protection-constants).

### `allocateNear(nearAddress: number, size: number, flProtect: number) -> number`

Allocates a block of memory with the specified size and protection flags near the specified address.
Useful when referencing from ASM scripts if a 32-bit offset is required.

### `deallocate(address: number)`

Deallocates a block of memory.

### `getModule(moduleName: string) -> number`

Returns the handle of the specified module (dll or exe).
See [MSDocs](https://learn.microsoft.com/en-us/windows/win32/api/libloaderapi/nf-libloaderapi-getmodulehandlea)

### `resolvePointer(pointer: string) -> number`

Resolves a multi-level pointer string to an address.
The pointer should be formatted as "base offset1 offset2 ... offsetN".
Base can be a module name or an address.

### `getProcAddress(module: number, procName: string) -> number`

Returns the address of the specified function in the specified module.
See [MSDocs](https://learn.microsoft.com/en-us/windows/win32/api/libloaderapi/nf-libloaderapi-getprocaddress)

### `executeAtomic(callback: function)`

Executes a callback atomically, suspending all other threads in the process.
Useful when creating hooks to avoid race conditions.

### `executeAtomicEx(callback: function, regionStart: number, regionSize: number)`

Executes a callback atomically, suspending all other threads in the process.
Avoids suspending threads inside a specified region.
```lua
-- Example: (From utils.createJumpHook)
executeAtomicEx(function()
    writeBytes(hook.address, originalCode)
    if hook.method == utils.hookMethods.int3 then
        removeInt3Jump(hook.address)
    end
end, hook.address, hook.nopCount)
```

### `hook(address: number, callback: function)`

Sets a breakpoint that triggers a Lua callback when hit.
Runs *before* the target instruction is executed.
Lua function can read/write register globals EAX/RAX, EBX,RBX...
```lua
-- Example:
hook(
    -- game.exe+68C410 - 8B 41 44 - mov eax,[rcx+44] ; Read true health value.
    -- game.exe+68C413 - C3       - ret 
    resolvePointer("game.exe 68C413"),
    function ()
        if getModPaused() then return end
        RAX = math.floor(RAX * getActivityLevelSmoothed() * 2)
    end
)
```

### `hookEx(address: number, callback: function, type: number)`

Sets a breakpoint that triggers a Lua callback when hit.
Lua function can read/write register globals EAX/RAX, EBX,RBX...

### `unhook(address: number)`

Removes a breakpoint set by hook or hookEx.

### `installInt3Jump(from: number, to: number)`

Installs an Int3 jump from the specified address to the target address.
Uses a vectored exception handler to catch the Int3 exception and redirect execution.

### `removeInt3Jump(from: number)`

Removes an Int3 jump from the specified address.

### `disassembleAt(address: number, length: number) -> string`

Disassembles the code at the specified address.

### `disassemble(bytes: table, runtimeAddress: number) -> string`

Disassembles the specified bytes as if they were at the specified runtime address.

### `relocate(address: number, newAddress: number) -> table`

Relocates a single instruction at the specified address to a new address.
Returns the relocated instruction bytes.
Useful when dealing with relative addressing or jumps.
This is not guaranteed to work for all instructions. Be sure to verify the relocated code.

### `relocateMultiple(address: number, newAddress: number, bytesToRelocate: number) -> table`

Relocates *atleast* the specified number of bytes starting from the specified address to a new address.
Returns the relocated instruction bytes.
Useful when dealing with relative addressing or jumps.
This is not guaranteed to work for all instructions. Be sure to verify the relocated code.
```lua
-- Example:
local relocatedBytes = relocateMultiple(runtimeAddress, tramplineAddress, bytesToRelocate)
writeBytes(tramplineAddress, relocatedBytes)
writeBytes(tramplineAddress + bytesToRelocate, yourAssembledCode)
```

### `jump5Byte(from: number, to: number) -> table`

Generates a 5-byte jump instruction from the specified address to the target address.
```lua
-- Example:
local jumpBytes = jump5Byte(runtimeAddress, tramplineAddress)
writeBytes(runtimeAddress, jumpBytes)
```

### `jump14Byte(address: number) -> table`

Generates a 14-byte jump instruction to the specified address.

### `assembleToBytes(runtimeAddress: number, symbols: table, code: string) -> table`

Assembles code to machine code bytes. Returns the assembled bytes.
Expects ASM code in Intel syntax.
Accepts a table of symbols which can be referenced in the ASM code.
```lua
-- Example:
local runtimeAddress = 0x400000
local symbols = { pPlayer = allocateNear(runtimeAddress, 4, PAGE_READWRITE) }
local code = [[
   cmp [eax+0x10], 0
   jne SKIP
   mov [pPlayer], eax ; This is the player, save it.
   SKIP:
   ret
]]
local bytes = assembleToBytes(runtimeAddress, symbols, code)
```

### `onTick(callback: fun(dtSeconds: number))`

Registers a callback for the onTick event.

### `onUnload(callback: fun())`

Registers a callback for the onUnload event.

### `onEffectStart(callback: fun(effectName: string))`

Registers a callback for the onEffectStart event.

### `onEffectEnd(callback: fun(effectName: string))`

Registers a callback for the onEffectEnd event.

### `onEffectTick(callback: fun(effectName: string, dtSeconds: number))`

Registers a callback for the onEffectTick event.
```lua
-- Example:
onEffectTick(function (effect, dt)
    if effect == "Stamina Burst" then
        addToVariable("Stamina", 200 * dt)
    elseif effect == "Health Burst" then
        addToVariable("Health", 100 * dt)
    end
end)
