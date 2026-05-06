# welcome

The orientation plugin. One skill plus one slash command, both pointing to the same calm two-minute map of the basic-harness marketplace.

This plugin does not bootstrap anything, write anything to disk, or run skills on your behalf. It's a map — read once, then move on to the plugin you actually want.

## Install

```text
/plugin marketplace add sureserverman/basic-harness
/plugin install welcome@basic-harness
```

## How to start the tour

**Three equivalent entry points** — all hit the same `marketplace-tour` skill:

1. **Natural language, in any language** — "show me what you do", "what can you do", "what is this", "introduce yourself", "что ты умеешь", "qué haces", "was kannst du", "qu'est-ce que tu fais", "你能做什么". The skill auto-fires.
2. **Slash command** — `/welcome:tour`. Explicit and stable.
3. **Phrasing the persona question directly** — "I'm a researcher / writer / project lead / personal user, where do I start?". The skill picks up and routes you to the relevant specialized tour.

## What's inside

| Component | What it does |
|---|---|
| `marketplace-tour` skill | Two-minute marketplace map. Auto-fires on natural-language questions ("show me what you do" and equivalents in any language). Same flow as the slash command. |
| `/welcome:tour` slash command | Thin wrapper around the same skill. Use when you want to invoke the tour explicitly without phrasing a question. |

## When to use it

- You just installed something from the basic-harness marketplace and want a calm read-only overview before doing anything else.
- A teammate sent you the marketplace and you want to know whether it fits your work before installing more plugins.
- You've been using one plugin for a while and want to see what else is in the bundle.

## When not to use it

- If you already know what you want to do, run that plugin's command directly.
- If you're a new personal-coach user, `/personal-coach:onboard` is a better starting point — it produces an actual artifact in five minutes, while this command only produces understanding.

## Multilingual

The `/welcome:tour` command runs in whatever language you write to it in. Same posture as `/personal-coach:onboard`.
