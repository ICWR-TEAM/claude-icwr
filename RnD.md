# ICWR Claude Plugin Marketplace

**Version:** 1.0.2 | **Author:** HarshXor

---

## Quick Start

1. **Create Structure**
   ```bash
   mkdir -p plugins/new-plugin/{.claude-plugin,commands,phases}
   ```

2. **Add plugin.json**
   ```json
   {
     "name": "new-plugin",
     "version": "1.0.0",
     "description": "Plugin description",
     "author": {"name": "Author", "email": "email"},
     "license": "MIT",
     "category": "development"
   }
   ```

3. **Create Command**
   - Add `commands/command.md` with usage

4. **Implement Code**
   - `01-implementation.md`: Main code
   - `02-testing.md`: Unit tests
   - `03-documentation.md`: Usage docs

5. **Update Registry**
   - Add to `.claude-plugin/marketplace.json`

---

## Categories

- **development**: Code generation, project setup
- **testing**: QA automation, test generation
- **security**: Vulnerability scanning, exploit tools
- **utility**: Code analysis, workflow tools

---

## Template

```python
import argparse

class ToolName:
    def __init__(self):
        self.parser = argparse.ArgumentParser(description="Tool description")
        self.setup_args()
        self.args = self.parser.parse_args()
        self.run()

    def setup_args(self):
        self.parser.add_argument('-t', '--target', required=True, help='Target')

    def run(self):
        print(f"Processing {self.args.target}")

if __name__ == "__main__":
    ToolName()
```

---

## Current Plugins

- **icwr-python-cli**: Python CLI tools (v1.0.0)

---

## Repository

https://github.com/ICWR-TEAM/claude-icwr