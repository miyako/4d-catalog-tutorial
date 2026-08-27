# 4D Catalog Tutorial

How to design a database schema in [4D](https://developer.4d.com/) using an AI coding agent.

## What This Repo Does

This repository contains instruction files that teach an AI agent how to generate a valid 4D database schema (`catalog.4DCatalog`) from a natural-language description of your business. The instructions live in `.github/instructions/` where GitHub Copilot and compatible tools automatically pick them up.

## Getting Started

### 1. Clone the repo

Open the repository in [GitHub Copilot](https://github.com/features/copilot) (or a similar AI coding tool that supports `.github/instructions/`).

```
gh repo clone miyako/4d-catalog-tutorial
```

### 2. Verify the agent can see the instructions

The instruction files are in:

```
.github/instructions/
├── 4d-project-instructions.md   # How to create a 4D project and catalog
└── tool4d-cli.md                # How to validate with tool4d (headless CLI)
```

Agents that support the `.github/instructions/` convention will load these automatically. If your tool requires explicit context, point it to these files.

### 3. Give the agent a prompt

Either pick one of the 50 example prompts included in `prompts/`:

```
prompts/
├── 000001.txt   # PedalPatch — mobile bicycle repair
├── 000002.txt   # ...
├── ...
└── 001030.txt
```

Or describe the database you want to create in your own words. For example:

> I'm opening a small bakery called SunriseBread. I need to track recipes, ingredients inventory with suppliers, daily production batches, retail customers, wholesale accounts, orders, and invoicing.

The agent will generate a complete 4D project structure including the `catalog.4DCatalog` XML schema with tables, fields, primary keys, indexes, and relations.

## What Gets Generated

A valid 4D project with:

- **Tables** with auto-increment primary keys and appropriate field types
- **Relations** (Many-to-One / One-to-Many) with ORDA navigation attributes
- **Indexes** (B-tree for PKs, Cluster for FKs)
- **Journal support** for data recovery

## Validating the Output

If you have [tool4d](https://developer.4d.com/docs/Admin/cli) installed, the agent can validate the generated project by running it headlessly. See `tool4d-cli.md` for details.

## References

- [4D Project Architecture](https://developer.4d.com/docs/Project/architecture)
- [4D Identifiers](https://developer.4d.com/docs/Concepts/identifiers)
- [ORDA Data Model Mapping](https://developer.4d.com/docs/ORDA/dsmapping)
- [tool4d CLI](https://developer.4d.com/docs/Admin/cli)
