# Nova Act Skill for OpenClaw

An [OpenClaw](https://github.com/openclaw/openclaw) skill for AI-powered browser automation using [Amazon Nova Act](https://github.com/aws/nova-act).

## Installation

Clone this repo to your OpenClaw skills directory:

```bash
git clone https://github.com/peterqliu/openclaw-nova-act ~/.openclaw/skills/nova-act
```

Or add to your `~/.openclaw/openclaw.json`:

```json
{
  "skills": {
    "load": {
      "extraDirs": ["~/workspace/openclaw-nova-act"]
    }
  }
}
```

## Requirements

- [uv](https://github.com/astral-sh/uv) - Python package manager
- `NOVA_ACT_API_KEY` environment variable set

## Usage

The skill provides a bundled script for quick browser automation:

```bash
uv run scripts/nova_act_runner.py \
    --url "https://google.com/flights" \
    --task "Find flights from SFO to NYC on March 15 and return the options"
```

Output is JSON with `summary` and `details` fields.

## Custom Scripts

For complex workflows, write custom Python scripts. See `SKILL.md` for API patterns and examples.

## License

MIT
