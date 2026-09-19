# luce-ai

AI support for the Luce editor ecosystem. This first module is the **configuration
layer**: which model answers each role, and how to reach each provider. It parses
and validates an `ai.toml` file (through [luce-config](../luce-config)) and holds
no network logic — credentials are stored, not resolved, so a key is read from the
environment only when a request is actually made.

Networking (OpenRouter over HTTP, the local Claude Code CLI) and the editor chat
and edit flows build on this and land in later modules.

## Configuration

Each **mode** (`ask`, `write`, `spellcheck`, `inline`) is its own table with a
`provider`, a `model` and an optional `system` prompt. `model` is the provider's
own id and may contain slashes (an OpenRouter id such as `deepseek/deepseek-chat`).

```toml
[ask]
provider = "openrouter"
model    = "deepseek/deepseek-chat"
system   = "You are a concise coding assistant."

[write]
provider = "openrouter"
model    = "anthropic/claude-sonnet"
system   = ""

[spellcheck]
provider = "openrouter"
model    = "openai/gpt-4o-mini"

[inline]
provider = "claude_code"
model    = "sonnet"

[openrouter]
api_key  = ""                       # a literal key, or leave empty and use key_env
key_env  = "OPENROUTER_API_KEY"     # an environment variable holding the key
base_url = "https://openrouter.ai/api/v1"

[claude_code]
command = "claude"                  # the local Claude Code CLI on PATH
```

`AiConfig(source)` validates the schema up front, like the editor's other settings:
unknown tables, unknown keys, and a mode naming an unknown provider are errors, not
silently ignored. `provider` defaults to `openrouter`; every mode is optional and
its model is unset by default.

- `mode(name)` returns the `Mode` (`provider`, `model`, `system`) for `ask` /
  `write` / `spellcheck` / `inline`, an empty Mode otherwise.
- `provider(name)` returns the `Provider` a mode resolves to.
- `ready(name)` is true when a mode names a model **and** that provider has a
  declared credential. It may still fail at request time if the environment
  variable is unset.

Table headers are flat (`[openrouter]`, not `[providers.openrouter]`) because the
shared TOML reader accepts only bare table names.

## Tests

```sh
./test.sh
```

Runs the module's test blocks natively and in the C comparison build.
