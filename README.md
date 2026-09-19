# luce-ai

AI support for the Luce editor ecosystem. This first module is the **configuration
layer**: which model answers each role, and how to reach each provider. It parses
and validates an `ai.toml` file (through [luce-config](../luce-config)) and holds
no network logic — credentials are stored, not resolved, so a key is read from the
environment only when a request is actually made.

Networking (OpenRouter over HTTP, the local Claude Code CLI) and the editor chat
and edit flows build on this and land in later modules.

## Configuration

```toml
[models]
# Each role is "provider/model". Providers: openrouter, claude_code.
ask        = "openrouter/anthropic/claude-opus"
write      = "openrouter/anthropic/claude-sonnet"
spellcheck = "openrouter/openai/gpt-4o-mini"
inline     = "claude_code/sonnet"

[openrouter]
api_key  = ""                       # a literal key, or leave empty and use key_env
key_env  = "OPENROUTER_API_KEY"     # an environment variable holding the key
base_url = "https://openrouter.ai/api/v1"

[claude_code]
command = "claude"                  # the local Claude Code CLI on PATH
```

`AiConfig(source)` validates the schema up front, like the editor's other settings:
unknown tables, unknown keys, and a role naming an unknown provider are errors, not
silently ignored. Every role is optional and unset by default.

- `role(name)` returns the raw `"provider/model"` for `ask` / `write` /
  `spellcheck` / `inline`, or `""`.
- `provider(name)` returns the `Provider` a role resolves to.
- `ready(name)` is true when a role names a model **and** that provider has a
  declared credential. It may still fail at request time if the environment
  variable is unset.
- `provider_of(value)` / `model_of(value)` split a `"provider/model"` string.

Table headers are flat (`[openrouter]`, not `[providers.openrouter]`) because the
shared TOML reader accepts only bare table names.

## Tests

```sh
./test.sh
```

Runs the module's test blocks natively and in the C comparison build.
