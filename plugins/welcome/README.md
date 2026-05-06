# welcome

The orientation plugin. One slash command, `/welcome:tour`, which gives a calm two-minute map of the basic-harness marketplace and points you at the right specialized tour for your work.

This plugin does not bootstrap anything, write anything to disk, or run skills on your behalf. It's a map — read once, then move on to the plugin you actually want.

## Install

```text
/plugin marketplace add sureserverman/basic-harness
/plugin install welcome@basic-harness
```

Then run:

```text
/welcome:tour
```

## What's inside

| Command | What it does |
|---|---|
| `/welcome:tour` | Two-minute marketplace map. What basic-harness is, the four personas, the install flow, and pointers to `/personal-coach:tour` (personal companion track) and `/process-skills:tour` (research / writing / project-management track). |

## When to use it

- You just installed something from the basic-harness marketplace and want a calm read-only overview before doing anything else.
- A teammate sent you the marketplace and you want to know whether it fits your work before installing more plugins.
- You've been using one plugin for a while and want to see what else is in the bundle.

## When not to use it

- If you already know what you want to do, run that plugin's command directly.
- If you're a new personal-coach user, `/personal-coach:onboard` is a better starting point — it produces an actual artifact in five minutes, while this command only produces understanding.

## Multilingual

The `/welcome:tour` command runs in whatever language you write to it in. Same posture as `/personal-coach:onboard`.
