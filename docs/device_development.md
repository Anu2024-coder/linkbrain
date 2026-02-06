# Device Development Guide

This document explains how to add new physical device abstractions to LinkBrain.

LinkBrain treats devices as **deterministic, stateful systems** with explicit contracts. Devices are not controlled by language models directly. Instead, they expose a limited, validated surface that the execution layer can safely operate on.

---

## Core concept

A device in LinkBrain represents a real physical component (light, fan, sensor, actuator) with:

* A known hardware interface
* A defined set of allowed actions
* Explicit state transitions
* Deterministic execution behavior

The device is responsible for **how** an action is executed. The AI is only responsible for **deciding which action to request**.

---

## Device layers

Every device integration spans two layers:

1. **Hardware device class** (execution)
2. **Tool wrapper** (AI-visible interface)

These layers must remain separate.

```
LLM → Tool → Device → Controller → Hardware
```

---

## 1. Hardware device class

The hardware device class lives under `linkbrain/devices/`.

Responsibilities:

* Translate high-level actions into controller calls
* Enforce device-specific constraints
* Track authoritative device state
* Raise explicit errors on failure

Example skeleton:

```python
class Light:
    def __init__(self, name: str, controller, pin: int):
        self.name = name
        self.controller = controller
        self.pin = pin
        self._state = "off"

    def on(self):
        self.controller.write_pin(self.pin, True)
        self._state = "on"

    def off(self):
        self.controller.write_pin(self.pin, False)
        self._state = "off"

    def status(self):
        return self._state
```

Rules:

* No LLM imports
* No prompt logic
* No free-form strings for execution

---

## 2. Tool wrapper

The tool wrapper lives under `linkbrain_core/tools/`.

Responsibilities:

* Declare allowed actions
* Validate action arguments
* Call device methods safely
* Return structured execution results

Example skeleton:

```python
class LightTool:
    def __init__(self, name: str, device: Light):
        self.name = name
        self.device = device

    async def execute(self, action: str):
        if action == "on":
            self.device.on()
        elif action == "off":
            self.device.off()
        elif action == "status":
            return {"state": self.device.status()}
        else:
            raise ValueError(f"Unsupported action: {action}")
```

Rules:

* Only expose whitelisted actions
* Reject unknown or malformed actions
* Never infer or guess intent

---

## Device contracts

Every device must have a clear contract:

* **Name** (unique identifier)
* **Type** (light, fan, sensor, etc.)
* **Allowed actions**
* **State model**

These contracts are surfaced to the AI via `DeviceContext`, not via code execution.

---

## State management

Device state must be:

* Owned by the device or controller
* Updated only after successful execution
* Queryable via explicit methods

Never allow the LLM to assume state based on conversation history.

---

## Error handling

Devices must fail loudly and explicitly.

Examples:

* Invalid pin
* Disconnected controller
* Unsupported action

Errors should:

* Be typed where possible
* Include clear messages
* Never be silently ignored

---

## Testing devices

All new devices must include tests.

Testing guidelines:

* Mock controller interfaces
* Test valid and invalid actions
* Verify state transitions
* Test failure paths

Live hardware should not be required for CI.

---

## Anti-patterns (do not do this)

* Calling hardware directly from LLM code
* Parsing natural language inside device classes
* Executing actions based on free-form text
* Hiding side effects inside tools

If your device requires these, rethink the design.

---

## Design checklist

Before submitting a device:

* [ ] Clear action contract
* [ ] Deterministic execution
* [ ] Explicit state handling
* [ ] Validation and errors
* [ ] Unit tests

---

## Final note

Device development is where safety is won or lost.

If a device abstraction feels flexible but unpredictable, it does not belong in LinkBrain.
