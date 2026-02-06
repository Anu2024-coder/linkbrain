# Contributing to LinkBrain

Thank you for your interest in contributing to LinkBrain. This project is building a deterministic control plane for AI-driven physical systems, and contributions are expected to meet a high bar for correctness, safety, and clarity.

This document outlines how to contribute effectively and what standards we expect.

---

## Project philosophy

LinkBrain is **not** a chatbot framework and **not** a generic IoT SDK. It is an execution runtime that sits between AI reasoning systems and physical hardware.

Core principles:

* Determinism over convenience
* Explicit contracts over implicit behavior
* Hardware safety over AI flexibility
* Clear system boundaries

All contributions should reinforce these principles.

---

## What you can contribute

We welcome contributions in the following areas:

* Core runtime stability and correctness
* New device abstractions (lights, fans, sensors, actuators)
* Communication layers (BLE, Wi‑Fi, Serial, MQTT)
* Structured parsing and execution improvements
* Testing, validation, and simulation tooling
* Documentation and architecture clarity

If you are unsure whether a change fits, open an issue first.

---

## What we do NOT accept

To keep the system safe and maintainable, we generally do not accept:

* Direct LLM → hardware execution paths
* Hidden side effects or implicit device actions
* Undocumented magic behavior
* Features that blur AI reasoning and hardware execution
* Changes that bypass validation or contracts

---

## Development setup

Clone the repository:

```bash
git clone https://github.com/satyam-tomar/linkbrain.git
cd linkbrain
```

Create a virtual environment and install dependencies:

```bash
pip install -e ".[dev]"
```

Run tests to ensure everything works:

```bash
pytest
```

---

## Code structure guidelines

* **linkbrain/**: hardware execution layer only
* **linkbrain_core/**: AI integration and reasoning support

Rules:

* Hardware code must never depend on LLM providers
* AI code must never call hardware directly
* All device actions must be explicit and validated

If a change breaks these rules, it will be rejected.

---

## Adding a new device

When adding a new device abstraction:

1. Define a clear device contract (actions, state, limits)
2. Implement deterministic execution methods
3. Expose only allowed actions via a Tool
4. Add unit tests for success and failure paths

Avoid embedding device logic inside prompts or parsers.

---

## Testing requirements

All non-trivial changes must include tests.

We expect:

* Unit tests for parsers, tools, and devices
* Failure-case coverage (invalid actions, disconnections)
* No reliance on live hardware for CI tests

Use mocks or simulation layers where needed.

---

## Coding standards

* Follow PEP8 and existing formatting
* Use type hints where applicable
* Prefer explicit logic over clever shortcuts
* Log meaningful execution steps

Before submitting:

```bash
black .
flake8 .
mypy .
pytest
```

---

## Submitting a pull request

1. Fork the repository
2. Create a feature branch
3. Make focused, atomic commits
4. Ensure tests pass
5. Open a pull request with a clear description

Your PR description should explain:

* What problem this solves
* Why the approach is safe and deterministic
* Any trade-offs introduced

---

## Reporting issues

When filing an issue, include:

* What you expected to happen
* What actually happened
* Logs or error messages
* Hardware details (if relevant)

Avoid vague bug reports.

---

## Community standards

Be respectful, precise, and constructive.

This project values:

* Engineering rigor
* Clear communication
* Safety-first thinking

If you’re here to push boundaries responsibly, you’re welcome.

---

## License

By contributing, you agree that your contributions will be licensed under the MIT License.
