# AGENTS.md — llm-cost

## Purpose

`llm-cost` is a zero-dependency, single-header C++ library for counting tokens
and estimating API costs before making LLM calls. Works offline — no network calls.

## Architecture

```
llm-cost/
  include/
    llm_cost.hpp        <- THE ENTIRE LIBRARY. Do not split this.
  examples/
    count_tokens.cpp    <- Count tokens + cost for a long prompt
    compare_models.cpp  <- Cost comparison table across all models
    budget_guard.cpp    <- Demonstrate assert_budget()
  CMakeLists.txt        <- Builds examples (no external deps needed)
```

## Build & Run

```bash
cmake -B build && cmake --build build
./build/count_tokens
./build/compare_models
./build/budget_guard
```

## Rules

- **Single header only.** `include/llm_cost.hpp` is the complete library.
- **No external dependencies.** Not even libcurl — this is pure offline computation.
- **All public API in namespace `llm`.**
- **Token counting is approximate** (±5% vs tiktoken). Do not claim exactness.
- **C++17, -Wall -Wextra, zero warnings.**
- **RAII for any resource handles** (none needed currently, but follow the pattern).

## Public API surface

```cpp
namespace llm {
    enum class Provider { OpenAI, Anthropic };
    struct Model { std::string name; Provider provider; double input_cost_per_1k,
                   output_cost_per_1k; size_t context_window; };

    namespace models {
        inline const Model GPT4O, GPT4O_MINI, GPT4_TURBO;
        inline const Model CLAUDE_OPUS, CLAUDE_SONNET, CLAUDE_HAIKU;
    }

    struct TokenCount { size_t tokens, characters; double estimated_cost_usd;
                        bool exceeds_context; std::string model_name; };
    struct CostComparison { std::string model_name; size_t tokens; double input_cost_usd; };

    TokenCount count(const std::string& text, const Model& model);
    TokenCount count_messages(const std::vector<std::pair<std::string,std::string>>&, const Model&);
    void assert_budget(const TokenCount& tc, double budget_usd);  // throws on exceed
    std::vector<CostComparison> compare_costs(const std::string& text);
    std::string format_cost(double usd);
}
```
