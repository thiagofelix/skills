# API Catalog

This catalog is derived from the merged OpenAPI spec at `~/.amazon-ads-openapi.json`.

## Tags

<!-- Generated list of tags from the OpenAPI spec -->

## Endpoints by Tag

<!-- Generated list of endpoints grouped by tag -->

## How to Regenerate

Run the following to extract tags and paths:

```bash
python - <<'PY'
import json
from collections import defaultdict

with open("/Users/thiagofelix/.amazon-ads-openapi.json") as f:
    spec = json.load(f)

tags = sorted({t for t in spec.get("tags", []) if isinstance(t, str)} | {t.get("name") for t in spec.get("tags", []) if isinstance(t, dict)})

by_tag = defaultdict(list)
for path, methods in spec.get("paths", {}).items():
    for method, op in methods.items():
        if not isinstance(op, dict):
            continue
        for tag in op.get("tags", ["untagged"]):
            by_tag[tag].append(f"{method.upper()} {path}")

print("## Tags\n")
for tag in tags:
    if tag:
        print(f"- {tag}")

print("\n## Endpoints by Tag\n")
for tag in sorted(by_tag.keys()):
    print(f"### {tag}\n")
    for entry in sorted(by_tag[tag]):
        print(f"- {entry}")
    print("")
PY
```
