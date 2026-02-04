---
name: nova-act
description: Write and execute Python scripts using Amazon Nova Act for AI-powered browser automation tasks like flight searches, data extraction, and form filling.
homepage: https://github.com/aws/nova-act
metadata:
  {
    "openclaw":
      {
        "emoji": "🌐",
        "requires": { "bins": ["uv"], "env": ["NOVA_ACT_API_KEY"] },
        "primaryEnv": "NOVA_ACT_API_KEY",
        "install":
          [
            {
              "id": "uv-brew",
              "kind": "brew",
              "formula": "uv",
              "bins": ["uv"],
              "label": "Install uv (brew)",
            },
          ],
        "tools":
          {
            "nova_act":
              {
                "description": "Run a browser automation task using Amazon Nova Act.",
                "parameters":
                  {
                    "type": "object",
                    "properties":
                      {
                        "url":
                          {
                            "type": "string",
                            "description": "Starting URL for the browser session",
                          },
                        "task":
                          {
                            "type": "string",
                            "description": "Natural language task description (e.g., 'Find flights from SFO to NYC')",
                          },
                      },
                    "required": ["url", "task"],
                  },
                "command":
                  [
                    "uv",
                    "run",
                    "{baseDir}/scripts/nova_act_runner.py",
                    "--url",
                    "{{url}}",
                    "--task",
                    "{{task}}",
                  ],
              },
          },
      },
  }
---

# Nova Act Browser Automation

Use Amazon Nova Act for AI-powered browser automation. The bundled script handles common tasks; write custom scripts for complex workflows.

## Quick Start with Bundled Script

Execute a browser task and get results:

```bash
uv run {baseDir}/scripts/nova_act_runner.py --url "https://google.com/flights" --task "Find flights from SFO to NYC on March 15 and return the options"
```

The script uses a generic schema (summary + details list) to capture output.

## Writing Custom Scripts

For complex multi-step workflows or specific extraction schemas, write a custom Python script with PEP 723 dependencies:

```python
#!/usr/bin/env python3
# /// script
# requires-python = ">=3.10"
# dependencies = ["nova-act"]
# ///

from nova_act import NovaAct

with NovaAct(starting_page="https://example.com") as nova:
    # Execute actions with natural language
    nova.act("Click the search box and type 'automation'")
    nova.act("Press Enter to search")

    # Extract data with schema
    results = nova.act_get(
        "Get the first 5 search result titles",
        schema=list[str]
    )
    print(results)

    # Take screenshot
    nova.page.screenshot(path="search_results.png")
    print(f"MEDIA: {Path('search_results.png').resolve()}")
```

Run with: `uv run script.py`

## Core API Patterns

### `nova.act(prompt)` - Execute Actions

Use for clicking, typing, scrolling, navigation:

```python
nova.act("Click the 'Sign In' button")
nova.act("Type 'hello@example.com' in the email field")
nova.act("Scroll down to the pricing section")
nova.act("Select 'California' from the state dropdown")
```

### `nova.act_get(prompt, schema)` - Extract Data

Use Pydantic models or Python types for structured extraction:

```python
from pydantic import BaseModel

class Flight(BaseModel):
    airline: str
    price: float
    departure: str
    arrival: str

# Extract single item
flight = nova.act_get("Get the cheapest flight details", schema=Flight)

# Extract list
flights = nova.act_get("Get all available flights", schema=list[Flight])

# Simple types
price = nova.act_get("What is the total price?", schema=float)
items = nova.act_get("List all product names", schema=list[str])
```

## Common Use Cases

### Flight Search

```python
with NovaAct(starting_page="https://google.com/flights") as nova:
    nova.act("Search for round-trip flights from SFO to JFK")
    nova.act("Set departure date to March 15, 2025")
    nova.act("Set return date to March 22, 2025")
    nova.act("Click Search")
    nova.act("Sort by price, lowest first")

    flights = nova.act_get(
        "Get the top 3 cheapest flights with airline, price, and times",
        schema=list[Flight]
    )
```

### Form Filling

```python
with NovaAct(starting_page="https://example.com/signup") as nova:
    nova.act("Fill the form: name 'John Doe', email 'john@example.com'")
    nova.act("Select 'United States' for country")
    nova.act("Check the 'I agree to terms' checkbox")
    nova.act("Click Submit")
```

### Data Extraction

```python
with NovaAct(starting_page="https://news.ycombinator.com") as nova:
    stories = nova.act_get(
        "Get the top 10 story titles and their point counts",
        schema=list[dict]  # Or use a Pydantic model
    )
```

## Best Practices

1. **Be specific in prompts**: "Click the blue 'Submit' button at the bottom" is better than "Click submit"
2. **Break complex tasks into steps**: Multiple `act()` calls are more reliable than one long instruction
3. **Use schemas for extraction**: Always provide a schema to `act_get()` for structured data
4. **Handle page loads**: Nova Act waits for stability, but add explicit waits for dynamic content if needed
5. **Take screenshots for verification**: Use `nova.page.screenshot()` to capture results

## API Key

- `NOVA_ACT_API_KEY` env var (required)
- Or set `skills."nova-act".apiKey` / `skills."nova-act".env.NOVA_ACT_API_KEY` in `~/.openclaw/openclaw.json`

## Notes

- Nova Act launches a real Chrome browser; ensure display is available or use headless mode
- The script prints `MEDIA:` lines for OpenClaw to auto-attach screenshots on supported providers
- For headless operation: `NovaAct(starting_page="...", headless=True)`
- Access underlying Playwright page via `nova.page` for advanced operations
