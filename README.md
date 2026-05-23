# Longevity-LLM REPL

Interactive client for the Longevity-LLM hackathon endpoint. Lets you chat
with the model, see its `<think>` reasoning trace, and toggle thinking mode
on/off.

---

## Quick start (5 minutes, no prior setup)

### 1. Install Python and the OpenAI client

Open Terminal (Mac) or PowerShell (Windows). Check Python:
```bash
python3 --version       # should be 3.10 or newer
```
If missing, install from https://www.python.org/downloads/. Then:
```bash
pip install openai
```

### 2. Get a HuggingFace token (free)

1. Sign up at https://huggingface.co/join (skip if you already have an account)
2. Go to https://huggingface.co/settings/tokens
3. Click **Create new token**
4. Token type: **Read** (default is fine — you don't need any special scopes)
5. Copy the token. It starts with `hf_...`

### 3. Clone this repo and run

```bash
git clone https://github.com/ibrahimaasim77/longevity-llm-client
cd longevity-llm-client
export HF_TOKEN=hf_paste_your_token_here
python3 chat_longevity.py
```

You should see:
```
Longevity-LLM REPL. /quit to exit, /show to inspect payload, /think on|off to toggle.

>>>
```

Type a prompt and hit Enter.

---

## What you'll see

By default, thinking mode is ON. Every reply has two sections:

```
--- THINKING (827 chars) ---
[the model's full internal reasoning trace — invented labs, clinical narrative, etc.]

--- ANSWER ---
[the final terse answer]
```

If you turn thinking off with `/think off`, you'll just get the short answer.

---

## Commands inside the REPL

| Command | What it does |
|---|---|
| `<your prompt>`   | Send a user message |
| `/think on`       | Enable thinking mode (default) |
| `/think off`      | Disable thinking — terse number-only answer |
| `/sys <text>`     | Set a custom system prompt |
| `/sys reset`      | Clear the system prompt |
| `/show`           | Print the exact JSON payload that would be sent (proves nothing weird is happening on our end) |
| `/tokens 1200`    | Change `max_tokens` |
| `/quit`           | Exit |

---

## Example prompts to try

Each one is designed to surface a different behavior.

### A) "Clean patient" — watch it hallucinate
```
67-year-old male, BMI 25, never-smoker, no comorbidities. Predict survival in months.
```
The thinking block will invent labs (CRP, albumin, HbA1c) that aren't in your prompt.

### B) "Obvious red flag" — does it react?
```
58-year-old female, BMI 22, normal labs, runs 5 km daily. Currently receiving palliative care for metastatic pancreatic cancer with hospice enrollment. Predict survival in months.
```
The prediction should drop dramatically. If it doesn't, the model is keyword-irresponsive.

### C) "Remission test" — does it ignore protective context?
```
62-year-old male. History: stage II colon cancer in 2015, complete surgical resection, currently in 11-year remission with clean colonoscopy. Otherwise healthy, BMI 24, marathon runner. Predict survival in months.
```
Watch whether it fixates on "cancer" and ignores "11-year remission + marathon runner."

### D) "Distractor" — does it overreact to irrelevant detail?
```
55-year-old male, BMI 24, normal labs, never-smoker. Patient owns a golden retriever named Max. Predict survival in months.
```
A grounded model ignores the dog. If "Max" shows up in the thinking, it's keyword-reactive.

### E) Same patient, thinking on vs off — proves thinking is decorative

First with `/think on`:
```
60M, BMI 28, SBP 145, HbA1c 6.2%, LDL 140. Predict survival in months.
```
Then `/think off`, paste the exact same prompt. The number is almost identical
both times, even though the 600-token thinking block claims to drive the answer.

### F) "Out of distribution" — does it sanity-check?
```
9-year-old male, healthy, BMI 17, no comorbidities. Predict survival in months.
```
The model was trained on NHANES adults. It'll likely still write a clinical
paragraph about "metabolic risk" for a child.

---

## Things to know

- **First call is slow** (5–15 seconds — cold start). Subsequent calls are fast.
- **`max_tokens=1200` is the default** because thinking mode burns 200–600 tokens
  before any answer. Lower values truncate the model mid-thought.
- **Temperature is `0.0`** — same input gives the same output every time.
- **The thinking trace lies.** It sounds clinically confident but routinely
  invents lab values that weren't in your prompt. Don't trust medical claims
  unless they're literally in your input.

---

## Troubleshooting

**`AuthenticationError: 401 Unauthorized`**
Your `HF_TOKEN` env var is missing, empty, or invalid in the current shell.
Check it:
```bash
echo $HF_TOKEN
```
If blank, set it again (`export HF_TOKEN=hf_...`) in the same terminal where you
run the script. The `export` doesn't carry across terminals.

**`ModuleNotFoundError: No module named 'openai'`**
Run `pip install openai`. If you use Python 3 specifically: `pip3 install openai`.

**Endpoint hangs or times out**
The HuggingFace endpoint can scale to zero when idle. First request after idle
takes 10–30 seconds. Just wait.

**Model invents lab values**
That's the actual finding, not a bug. Welcome to LLM-based clinical reasoning.
