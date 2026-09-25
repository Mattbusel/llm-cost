# llm-cost

[![CI](https://github.com/Mattbusel/llm-cost/actions/workflows/ci.yml/badge.svg)](https://github.com/Mattbusel/llm-cost/actions/workflows/ci.yml)
[![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
![C++17](https://img.shields.io/badge/C%2B%2B-17-blue.svg)
![Single header](https://img.shields.io/badge/single-header-green.svg)

Estimate tokens and dollar cost for OpenAI and Anthropic models before you send a request, in one dependency-free C++ header.

> Part of **[llm-cpp](https://github.com/Mattbusel/llm-cpp)**, a family of 26 single-header C++ libraries for building on LLM APIs. Each one stands alone: copy one header, include it, done.

Surprise API bills usually come from prompts that were bigger than anyone thought. llm-cost gives you a quick offline token estimate and input cost for a prompt, a cheapest-first comparison across models, and a budget check you can put in front of every call. No network, no tokenizer files.

## Features

- Token estimate for a string or a chat message list (adds per-message overhead the way OpenAI counts it)
- Input cost in USD and a flag when the prompt exceeds the model's context window
- Six built-in models: `gpt-4o`, `gpt-4o-mini`, `gpt-4-turbo`, `claude-opus-4-5`, `claude-sonnet-4-5`, `claude-haiku-4-5`
- `compare_costs()`: the same prompt priced across all built-in models, cheapest first
- `assert_budget()`: throw before a call that would cost more than your limit
- `format_cost()`: human-readable dollars or cents
- Define your own `llm::Model` for any other model and price

## Quick start

Requirements: a C++17 compiler. No other dependencies, no network access.

1. Copy [`include/llm_cost.hpp`](include/llm_cost.hpp) into your project.
2. In exactly one `.cpp` file, `#define LLM_COST_IMPLEMENTATION` before including it. Other files just `#include "llm_cost.hpp"`.

```cpp
#define LLM_COST_IMPLEMENTATION
#include "llm_cost.hpp"
#include <iostream>

int main() {
    std::string prompt = "Explain the difference between a mutex and a semaphore.";

    llm::TokenCount tc = llm::count(prompt, llm::models::GPT4O_MINI);
    std::cout << tc.tokens << " tokens, " << llm::format_cost(tc.estimated_cost_usd) << "\n";

    // Cheapest first, across the six built-in models
    for (const auto& c : llm::compare_costs(prompt))
        std::cout << c.model_name << ": " << llm::format_cost(c.input_cost_usd) << "\n";

    llm::assert_budget(tc, 0.01);  // throws std::runtime_error if the input would cost more than $0.01
}
```

Build and run:

```bash
g++ -std=c++17 -I include example.cpp -o example
./example
```

Output:

```text
22 tokens, 0.0003¢
gpt-4o-mini: 0.0003¢
claude-haiku-4-5: 0.0006¢
claude-sonnet-4-5: 0.0066¢
gpt-4o: 0.0110¢
gpt-4-turbo: 0.0220¢
claude-opus-4-5: 0.0330¢
```

## API

Everything lives in namespace `llm`.

| Function / type | What it does |
|---|---|
| `count(text, model)` | Return a `TokenCount` (tokens, characters, estimated input cost, context overflow flag) |
| `count_messages(messages, model)` | Same for a list of `{role, content}` pairs |
| `compare_costs(text)` | Price the text on every built-in model, sorted cheapest first |
| `assert_budget(tc, budget_usd)` | Throw `std::runtime_error` if the estimate exceeds the budget |
| `format_cost(usd)` | Format a cost for display |
| `llm::models::*` | Built-in `Model` constants (name, provider, input/output price per 1K tokens, context window) |

## How it works

Token counts come from a cl100k_base-style heuristic: roughly four characters per token for prose, adjusted for code, numbers and non-ASCII text. The header's own documentation targets about 5% of tiktoken. Cost is tokens times the model's input price per 1K tokens.

## Examples

The [`examples/`](examples) folder has runnable programs:

- [`budget_guard.cpp`](examples/budget_guard.cpp)
- [`compare_models.cpp`](examples/compare_models.cpp)
- [`count_tokens.cpp`](examples/count_tokens.cpp)

Build the examples with CMake:

```bash
cmake -B build
cmake --build build
```

## Limitations

- Token counts are estimates, not exact tokenizer output, and the same heuristic is used for Anthropic models.
- Prices are hardcoded in the header and change over time; check them against the providers' current pricing or define your own `Model`.
- Only input cost is estimated, since output length is unknown before the call.

## License

MIT. See [LICENSE](LICENSE).
