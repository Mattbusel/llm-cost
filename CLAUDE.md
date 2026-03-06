# CLAUDE.md — llm-cost

## Build

```bash
cmake -B build && cmake --build build
```

## Run examples

```bash
./build/count_tokens
./build/compare_models
./build/budget_guard
```

## Key constraint: SINGLE HEADER

`include/llm_cost.hpp` is the entire library. Never split into multiple files.
Never create a `src/` directory. The `#ifdef LLM_COST_IMPLEMENTATION` guard is
intentional — define it in exactly one translation unit.

## No network calls

This library is purely offline. Do not add libcurl or any HTTP code. Token
counting is a local heuristic; cost calculation is pure arithmetic.

## Token counting accuracy

The cl100k_base approximation in `detail::estimate_tokens()` is ±5% vs tiktoken
on English prose, ±10% on code. This is intentional — exact BPE requires the
full tiktoken vocabulary table (~2MB). The heuristic is good enough for budget
guards and cost estimates. Do not attempt to embed the BPE table.

## Public API to maintain

```cpp
namespace llm {
    TokenCount count(const std::string& text, const Model& model);
    TokenCount count_messages(const std::vector<std::pair<std::string,std::string>>&, const Model&);
    void assert_budget(const TokenCount& tc, double budget_usd);
    std::vector<CostComparison> compare_costs(const std::string& text);
    std::string format_cost(double usd);
}
```

## Common mistakes to avoid

- Adding tiktoken, sentencepiece, or any tokenizer as a dep — not allowed
- Returning exact token counts — always document as estimates
- Forgetting `#ifdef LLM_COST_IMPLEMENTATION` around the implementation block
- Throwing from inside token-counting functions (only `assert_budget` throws)
