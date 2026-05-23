# Longevity-LLM REPL

Minimal interactive client for the Longevity-LLM hackathon endpoint. Lets you
chat with the model, see its `<think>` block, and toggle thinking mode on/off.

## Setup

```bash
pip install openai
export HF_TOKEN=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   # token provided by the hackathon organizers
python3 chat_longevity.py
```

If you skip `export`, the script will prompt you for the token interactively.

## Commands inside the REPL

| command | what it does |
|---|---|
| `>>> <your prompt>`  | send a user message |
| `/think on`          | enable the `<think>` reasoning trace (default) |
| `/think off`         | terse answer only, no thinking |
| `/sys <text>`        | set the system prompt |
| `/sys reset`         | clear the system prompt |
| `/show`              | print the exact JSON payload that would be sent |
| `/tokens 1200`       | change max_tokens |
| `/quit`              | exit |

## Notes

- First call is slow (cold start); subsequent calls are fast.
- `max_tokens=1200` is the default because thinking mode burns 200–600 tokens
  before any answer. Lower values truncate the model mid-thought.
- Temperature is `0.0` — same input gives the same output.
- The model's thinking trace often invents lab values that weren't in your
  prompt. Don't trust medical claims that aren't grounded in your input.
