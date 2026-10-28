# LLM Call Chain

Practice notebook for **prompt chaining**: calling an LLM several times in a row, where the output of one call becomes the input of the next. The model runs on **Groq** (`openai/gpt-oss-20b`).

## What's inside

- **`llm_call(prompt)`**: a small wrapper around one Groq chat call. It sends a prompt, returns the text of the reply, and keeps a running count of input, output and total tokens.
- **`writing_improvement_chain(topic)`**: a 3-step chain.
  1. Write a first draft paragraph about the topic.
  2. Critique the draft (3 specific improvements, without rewriting it).
  3. Rewrite the draft using the critique.
  
  Returns a dictionary with `draft`, `critique` and `final`.
- **`summarization_chain(topic)`**: a 3-step chain that condenses information.
  1. Explain the topic in 3 paragraphs.
  2. Summarize the explanation in exactly 3 sentences.
  3. Condense the summary into a tweet of 280 characters or less, ending with a hashtag.
  
  Returns a dictionary with `explanation`, `summary` and `tweet`.

## How a chain works

```
topic  ->  llm_call  ->  explanation  ->  llm_call  ->  summary  ->  llm_call  ->  tweet
```

Each step is a separate model call. The result of a step is placed into the prompt of the next one, so the work is split into small, checkable pieces.

## Setup

```bash
git clone https://github.com/razangit5-pixel/llm_call_chain.git
cd llm_call_chain
pip install groq python-dotenv
```

Create a `.env` file in the project folder:

```
GROQ_API_KEY=your_groq_key
```

Never commit `.env`. Make sure it is listed in `.gitignore`.

## Usage

Open the notebook and run the cells in order. The setup cell creates the Groq client, then you can try a chain:

```python
result = summarization_chain("quantum computing")

print(result["summary"])
print(result["tweet"])
print(f"Input: {input_tokens}, Output: {output_tokens}, Total: {total_tokens_used}")
```

Reset the token counters to `0` before a run if you want the count for that run only.

## Notes and lessons learned

- **`gpt-oss-20b` is a reasoning model.** It writes hidden "thinking" tokens before the answer, and those count toward `max_completion_tokens`. If the limit is too small, the reply can come back **empty**. Use `reasoning_effort="low"` and a generous `max_completion_tokens` (for example 4096, or 8192 for harder steps).
- **`max_completion_tokens` limits output only.** Input (prompt) tokens do not count toward it, but input plus output must fit in the model's context window.
- **Models are bad at counting.** Asking for "280 characters" or "exactly 3 sentences" is a request, not a guarantee. Asking for a **word limit** works better than a character limit, and a quick check in code (`len(tweet)`, a sentence count) catches the rest.
- **Add code checks only for rules that must never break**, such as a hard length limit. Let the prompt handle everything else.

## Tech stack

- Python
- Groq API (`openai/gpt-oss-20b`)
- python-dotenv
- Jupyter Notebook
