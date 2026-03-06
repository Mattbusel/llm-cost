# llm-cost

Count tokens and estimate API costs before you send. Zero deps, one header.

![C++17](https://img.shields.io/badge/C%2B%2B-17-blue)
![License: MIT](https://img.shields.io/badge/License-MIT-green)
![Single Header](https://img.shields.io/badge/single-header-orange)
![No Network](https://img.shields.io/badge/network-none-lightgrey)

---

## 30-second quickstart

```cpp
#define LLM_COST_IMPLEMENTATION
#include "llm_cost.hpp"

#include <iostream>

int main() {
    std::string prompt = "Explain distributed consensus in detail...";

    llm::TokenCount tc = llm::count(prompt, llm::models::GPT4O_MINI);

    std::cout << tc.tokens << " tokens, cost: "
              << llm::format_cost(tc.estimated_cost_usd) << "\n";

    // Block if over budget
    llm::assert_budget(tc, 0.01); // throws if > $0.01
}
```

---

## Installation

Copy `include/llm_cost.hpp` into your project. No other files needed.

```bash
cp llm_cost.hpp /your/project/include/
```

In **one** `.cpp` file:
```cpp
#define LLM_COST_IMPLEMENTATION
#include "llm_cost.hpp"
```

In all other files:
```cpp
#include "llm_cost.hpp"
```

No link flags needed — this library makes zero network calls and has no external dependencies.

---

## API reference

### `llm::count` — single string

```cpp
llm::TokenCount tc = llm::count(text, llm::models::GPT4O);
// tc.tokens             — estimated token count
// tc.characters         — raw character count
// tc.estimated_cost_usd — input-side USD cost
// tc.exceeds_context    — true if tokens > model's context window
// tc.model_name         — model identifier string
```

### `llm::count_messages` — chat format

```cpp
std::vector<std::pair<std::string,std::string>> msgs = {
    {"system", "You are helpful."},
    {"user",   "Explain recursion."},
};
llm::TokenCount tc = llm::count_messages(msgs, llm::models::CLAUDE_SONNET);
```

Includes per-message overhead tokens (role + delimiters), matching OpenAI's counting method.

### `llm::assert_budget` — enforce cost limits

```cpp
llm::assert_budget(tc, 0.05); // throws std::runtime_error if estimated cost > $0.05
```

Call this before your API call to gate expensive requests.

### `llm::compare_costs` — cross-model comparison

```cpp
auto rows = llm::compare_costs("Your prompt here");
for (auto& r : rows)
    std::cout << r.model_name << ": " << llm::format_cost(r.input_cost_usd) << "\n";
// Sorted cheapest-first
```

### `llm::format_cost` — human-readable pricing

```cpp
llm::format_cost(0.00034)  // → "0.0340¢"
llm::format_cost(0.025)    // → "$0.0250"
```

---

## Model pricing table

| Model | Provider | Input (per 1K tok) | Output (per 1K tok) | Context |
|-------|----------|--------------------|---------------------|---------|
| `gpt-4o` | OpenAI | $0.00500 | $0.01500 | 128K |
| `gpt-4o-mini` | OpenAI | $0.00015 | $0.00060 | 128K |
| `gpt-4-turbo` | OpenAI | $0.01000 | $0.03000 | 128K |
| `claude-opus-4-5` | Anthropic | $0.01500 | $0.07500 | 200K |
| `claude-sonnet-4-5` | Anthropic | $0.00300 | $0.01500 | 200K |
| `claude-haiku-4-5` | Anthropic | $0.00025 | $0.00125 | 200K |

Prices as of early 2026. Check provider dashboards for current rates.

---

## Token counting accuracy

**Token counts are estimates, not exact values.**

This library implements a cl100k_base approximation without requiring the full
BPE vocabulary table. Accuracy:

| Content type | Typical accuracy vs tiktoken |
|---|---|
| English prose | ±3–5% |
| Mixed code | ±5–10% |
| Non-ASCII / Unicode | ±10–15% |

For exact counts, use [tiktoken](https://github.com/openai/tiktoken) (Python) or
count from the API response's `usage` field after the call. This library is
designed for **pre-call budget checks**, where ±5% is sufficient.

---

## Examples

| File | Description |
|------|-------------|
| [`examples/count_tokens.cpp`](examples/count_tokens.cpp) | Count a long prompt, show input + estimated output cost |
| [`examples/compare_models.cpp`](examples/compare_models.cpp) | Print cost table for same prompt across all models |
| [`examples/budget_guard.cpp`](examples/budget_guard.cpp) | Demonstrate `assert_budget` blocking expensive calls |

## Building examples

```bash
cmake -B build
cmake --build build
./build/count_tokens
./build/compare_models
./build/budget_guard
```

---

## Why

- **Avoid surprise bills.** Gate expensive API calls with a one-liner budget check.
- **Pick the right model.** `compare_costs()` shows you the cheapest model for each prompt at a glance.
- **No runtime overhead.** Pure C++17 arithmetic — runs in microseconds, zero allocations on the hot path.

---

## See Also

`llm-cost` is part of the **C++ LLM Toolkit Suite** — single-header libraries that
compose into a full LLM integration stack:

| Repo | What it does |
|------|-------------|
| [llm-stream](https://github.com/Mattbusel/llm-stream) | Stream OpenAI & Anthropic responses via SSE |
| **llm-cost** | Token counting + cost estimation (this repo) |
| [llm-cache](https://github.com/Mattbusel/llm-cache) | Exact & semantic response caching |
| [llm-retry](https://github.com/Mattbusel/llm-retry) | Retry logic + circuit breaker |
| [llm-format](https://github.com/Mattbusel/llm-format) | Structured output / JSON schema enforcement |

All libraries follow the same conventions: single header, C++17, `namespace llm`,
`#ifdef *_IMPLEMENTATION` guard, zero heavy dependencies.

---

## License

MIT — see [LICENSE](LICENSE).
