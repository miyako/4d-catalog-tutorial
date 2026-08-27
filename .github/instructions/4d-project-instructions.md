---
object: "4d-project"
json_type: null
requires: ["tool4d-cli.md"]
keywords: ["4D", "project", "catalog", "database", "schema", "table", "field", "relation", "index", "primary key", "ORDA", "XML", "catalog.4DCatalog"]
summary: "How to create a 4D database application project from scratch: project structure, catalog schema (XML), tables, fields, primary keys, indexes, relations, and naming rules."
---

# Creating a 4D Database Application Project

Reference: https://developer.4d.com/docs/Project/architecture

## Project Structure

Given a project named `{ProjectName}`, create the following structure:

```
{ProjectName}/
├── .gitignore
├── Project/
│   ├── {ProjectName}.4DProject
│   └── Sources/
│       ├── catalog.4DCatalog          # Database schema (XML)
│       ├── catalog_editor.json        # Visual layout for IDE (JSON)
│       ├── folders.json               # Explorer folder definitions
│       ├── menus.json                 # Menu bar definitions
│       └── roles.json                 # Security/permissions
└── Resources/
```

## File Contents

### .gitignore

```
# Data folder
[dD][aA][tT][aA]/

# Derived data
DerivedData/

# Libraries
Libraries/

# User preferences
userPreferences.*/

# Uncomment (remove the following # ) to ignore Trash
# Project/Trash

# System files
.DS_Store
ehthumbs.db
Thumbs.db
```

### {ProjectName}.4DProject

```json
{
  "$comment": "The project file serves as an anchor to locate other project files",
  "compatibilityVersion": 2130,
  "tokenizedText": false
}
```

- `compatibilityVersion`: 4D version code. `2130` = 4D 21 R3. Encoding: LTS = `major*100+minor` (e.g., 21.3 = 2103), Feature release = `major*100+R_number*10` (e.g., 21 R3 = 2130).
- `tokenizedText`: Set to `false` when the project is created/maintained by an agent (not the IDE). Tokens like `:C10` or `:5` are only useful in IDE workflows. When `false`, write **plain 4D code without token suffixes** in all `.4dm` files.

### catalog_editor.json

Stores visual/graphical data for the IDE structure editor (table positions, colors, field colors).
For agent-generated projects, use a minimal version:

```json
{
  "tables": {}
}
```

### folders.json

```json
{
  "Default Classes": {},
  "Default Folder": {
    "groups": [
      "Default Classes",
      "Default Forms",
      "Default Project Methods",
      "Default Tables"
    ]
  },
  "Default Forms": {},
  "Default Project Methods": {},
  "Default Tables": {},
  "trash": {}
}
```

### menus.json

Default menu bar with File (Quit), Edit (standard items), and Mode (Design):

```json
{
  "bars": [
    {
      "id": 1,
      "name": "Barre #1",
      "items": [
        { "link": 32001 },
        { "link": 32002 },
        { "link": 32003 }
      ]
    }
  ],
  "menus": [
    {
      "link": 32001,
      "title": ":xliff:CommonMenuFile",
      "items": [
        { "title": ":xliff:CommonMenuItemQuit", "shortcutAccel": true, "shortcutKey": "Q", "action": "quit" }
      ]
    },
    {
      "link": 32002,
      "title": ":xliff:CommonMenuEdit",
      "items": [
        { "title": ":xliff:CommonMenuItemUndo", "shortcutAccel": true, "shortcutKey": "Z", "action": "undo" },
        { "title": "(-", "isSeparator": true },
        { "title": ":xliff:CommonMenuItemCut", "shortcutAccel": true, "shortcutKey": "X", "action": "cut" },
        { "title": ":xliff:CommonMenuItemCopy", "shortcutAccel": true, "shortcutKey": "C", "action": "copy" },
        { "title": ":xliff:CommonMenuItemPaste", "shortcutAccel": true, "shortcutKey": "V", "action": "paste" },
        { "title": ":xliff:CommonMenuItemClear", "action": "clear" },
        { "title": ":xliff:CommonMenuItemSelectAll", "shortcutAccel": true, "shortcutKey": "A", "action": "selectAll" },
        { "title": "(-", "isSeparator": true },
        { "title": ":xliff:CommonMenuItemShowClipboard", "action": "showClipboard" }
      ]
    },
    {
      "link": 32003,
      "title": ":xliff:CommonMenuMode",
      "items": [
        { "title": ":xliff:CommonMenuItemDesign", "action": "design" }
      ]
    }
  ]
}
```

### roles.json

```json
{
  "forceLogin": false,
  "restrictedByDefault": false,
  "privileges": [],
  "roles": [],
  "permissions": {
    "allowed": [
      {
        "applyTo": "ds",
        "type": "datastore",
        "read": [],
        "create": [],
        "update": [],
        "drop": [],
        "execute": [],
        "promote": []
      }
    ]
  }
}
```

## catalog.4DCatalog — Database Schema (XML)

The core schema file defining tables, fields, primary keys, indexes, and relations.

### Basic Structure

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE base SYSTEM "http://www.4d.com/dtd/2007/base.dtd" >
<base name="{ProjectName}" uuid="{BASE_UUID}" collation_locale="en-gb">
  <schema name="DEFAULT_SCHEMA"/>

  <!-- Tables go here -->

  <!-- Relations go here (after all tables, before indexes) -->

  <!-- Indexes go here (after relations, before base_extra) -->

  <base_extra>
    <journal_file journal_file_enabled="true"/>
  </base_extra>
</base>
```

- `journal_file_enabled="true"`: Enables the log/journal file for data recovery. Requires each table to have a single-field primary key.

### UUIDs

All structural elements (tables, fields, indexes, relations) require a `uuid` attribute — a 32-character hex string (only digits 0-9 and letters A-F; case-insensitive; stored as 128-bit integer).

**Rules:**
- UUIDs must be **unique within the catalog** and **referentially consistent** (e.g., `<primary_key field_uuid="X">` must match the field's `uuid="X"`)
- UUIDs can be **deterministic** — no need for true random UUIDs
- Recommended scheme for agents — encode resource type and IDs (use only hex digits 0-9, A-F):
  - Base: `B000...`
  - Tables: `A{table_id padded}...` (32 chars total, zero-padded)
  - Fields: `F{table_id}{field_id}...`
  - Indexes: `C{table_id}{field_id}...`
  - Relations: `D{relation_number}...`

Example scheme (zero-padded to 32 hex chars, using only hex digits 0-9 A-F):
```
Base:       B0000000000000000000000000000000
Table 1:    A0010000000000000000000000000000
Table 2:    A0020000000000000000000000000000
Field 1.1:  F0010001000000000000000000000000
Field 1.2:  F0010002000000000000000000000000
Field 2.1:  F0020001000000000000000000000000
Index 1.1:  C0010001000000000000000000000000
Relation 1: D0010000000000000000000000000000
```

Prefix legend (all valid hex): `A` = table, `B` = base, `C` = index, `D` = relation, `E` = reserved, `F` = field.

### Naming Rules for Tables and Fields

Reference: https://developer.4d.com/docs/Concepts/identifiers

**Constraints:**
- Max **31 characters**
- Must begin with: **letter, underscore, or dollar sign** (`$`)
- Can contain: letters, digits, underscores, spaces (but avoid spaces — see below)
- **Case insensitive** — `Name` and `name` are the same identifier
- For SQL compatibility: only `_0123456789abcdefghijklmnopqrstuvwxyz` — no spaces

**CRITICAL — Avoid reserved names as table or field names (case-insensitive match).**

Common traps that agents fall into: `Type`, `Date`, `Time`, `String`, `File`, `Folder`, `Form`, `Field`, `Table`, `Session`, `Storage`, `Log`, `Length`, `Position`, `Null`, `True`, `False`, `Not`, `Old`, `Self`, `This`, `Super`, `New`, `Query`, `Formula`, `Sum`, `Average`, `Max`, `Min`, `Int`, `Num`, `Round`, `Mod`, `Char`, `Level`, `Timestamp`.

Instead of `type` → use `itemType`, `penType`, `inkType`, etc. Instead of `date` → use `orderDate`, `createdDate`, etc. Instead of `string` → use `labelText`, etc.

Full list of reserved single-word 4D commands:
`Sum`, `Average`, `Max`, `Min`, `Int`, `Dec`, `String`, `Date`, `Time`, `Type`, `Length`, `Position`, `Num`, `Abs`, `Exp`, `Log`, `Cos`, `Sin`, `Tan`, `Mod`, `Random`, `Round`, `Trunc`, `Char`, `Lowercase`, `Uppercase`, `Substring`, `Bool`, `Choose`, `Not`, `Old`, `Modified`, `Locked`, `Self`, `This`, `Super`, `Form`, `File`, `Folder`, `Table`, `Field`, `Semaphore`, `Session`, `Storage`, `Null`, `True`, `False`, `Undefined`, `Timestamp`, `Level`, `Keystroke`, `Variance`, `Formula`, `ABORT`, `ACCEPT`, `ALERT`, `ASSERT`, `BACKUP`, `BEEP`, `CANCEL`, `CONFIRM`, `DIALOG`, `IDLE`, `MESSAGE`, `PLAY`, `QUERY`, `REDRAW`, `REJECT`, `RESTORE`, `TRACE`, `cs`, `ds`, `throw`

Also avoid 4D keywords: `If`, `Else`, `End if`, `For`, `While`, `Repeat`, `Until`, `Case of`, `End case`, `Return`, `Break`, `Continue`, `var`, `New`

**Recommended conventions:**
- **Tables**: PascalCase, plural (e.g., `Persons`, `Companies`, `OrderItems`)
- **Fields**: camelCase (e.g., `firstName`, `lastName`, `companyID`)
- **No spaces** — breaks ORDA dot notation and SQL
- Don't give a table the same name as a class (check `Project/Sources/Classes/` for existing `.4dm` files — comparison is **case-insensitive**)
- **Relation names must not clash with field names** in the same table — both become ORDA attributes (e.g., if a table has field `company`, don't name a relation `name_Nto1="company"`)

### Field Types

| Code | Type | Notes |
|------|------|-------|
| `1` | Boolean | TRUE/FALSE |
| `3` | Integer (16-bit) | -32,768 to 32,767 |
| `4` | Longint (32-bit) | ±2,147,483,647 — **standard for primary keys** |
| `5` | Integer 64-bit | SQL engine only; converted to Real in 4D language |
| `6` | Real | ±1.7E±308, 13 significant digits; don't use for identifiers |
| `8` | Date | Year 100 to 32,767 |
| `9` | Time | Stored as seconds |
| `10` | Alpha (string) | 1–255 chars; **requires `limiting_length` attribute** |
| `12` | Picture | Stored outside records; supports EXIF keyword indexing |
| `14` | Text | Up to 2 GB; B-tree indexes only cover first 1024 chars |
| `18` | BLOB | Up to 2 GB; binary data |
| `21` | Object | JSON key/value pairs; can contain nested objects, collections |

**Invalid type codes** (will cause 4D to reject/strip the field): `2`, `7`, `11`, `13`, `15`, `16`, `17`, `19`, `20`. Always use the codes listed above.

### Field Attributes

Common attributes on `<field>`:
- `name` (required): Field name
- `uuid` (required): Unique identifier
- `type` (required): Numeric type code
- `id` (required): Sequential field number within the table (starting from 1)
- `limiting_length="N"`: **Required for Alpha (type 10)** — max chars (1–255)
- `unique="true"`: Enforce uniqueness
- `autosequence="true"`: Auto-increment (for Longint primary keys)
- `autogenerate="true"`: Auto-generate value (for UUID primary keys)
- `not_null="true"`: Reject NULL values
- `never_null="true"`: Map NULL to blank/zero (4D language compatibility)
- `store_as_UUID="true"`: Store Alpha field as UUID format
- `outside_blob="true"`: Store data outside the record file
- `hide_in_REST="true"`: Hide from REST/ORDA exposure

### Primary Keys

Every table **should** have a single-field primary key for:
- ORDA access (required — tables without single-field PK are invisible to ORDA)
- Journal/log file support (required — `prevent_journaling="true"` needed without it)

**Longint auto-increment PK (recommended):**
```xml
<field name="ID" uuid="{UUID}" type="4" unique="true" autosequence="true" not_null="true" id="1"/>
<primary_key field_name="ID" field_uuid="{UUID}"/>
```

**UUID PK (alternative):**
```xml
<field name="ID" uuid="{UUID}" type="10" limiting_length="255" unique="true" autogenerate="true" store_as_UUID="true" not_null="true" id="1"/>
<primary_key field_name="ID" field_uuid="{UUID}"/>
```

### Tables

```xml
<table name="TableName" uuid="{TABLE_UUID}" id="{TABLE_NUMBER}">
  <!-- fields -->
  <primary_key field_name="ID" field_uuid="{FIELD_UUID}"/>
</table>
```

- `id`: Sequential table number starting from 1
- `prevent_journaling="true"`: Add this if the table has no single-field primary key
- `hide_in_REST="true"`: Hide from REST API

### Indexes

Indexes are defined as top-level elements (siblings of `<table>`, before `<base_extra>`).

**Index type values:**
| Type | Meaning | Use for |
|------|---------|---------|
| `1` | B-tree | Primary keys, unique fields, composite indexes |
| `3` | Cluster B-tree | Foreign keys, booleans, low-cardinality fields |
| `7` | Automatic | **Default — safe choice for agents** |
| `8` | Automatic for Object | Only for Object (type 21) fields |

**Index kind values:**
| Kind | Use for |
|------|---------|
| `regular` | Standard and composite indexes |
| `keywords` | Full-text search (Alpha, Text, Picture fields) |

**Primary key index (always required with PK):**
```xml
<index kind="regular" unique_keys="true" uuid="{INDEX_UUID}" type="7">
  <field_ref uuid="{FIELD_UUID}" name="ID">
    <table_ref uuid="{TABLE_UUID}" name="TableName"/>
  </field_ref>
</index>
```

**Foreign key index:**
```xml
<index kind="regular" unique_keys="false" uuid="{INDEX_UUID}" type="3">
  <field_ref uuid="{FIELD_UUID}" name="foreignKeyField">
    <table_ref uuid="{TABLE_UUID}" name="TableName"/>
  </field_ref>
</index>
```

**Keyword index (for text search):**
```xml
<index kind="keywords" unique_keys="false" uuid="{INDEX_UUID}" type="7">
  <field_ref uuid="{FIELD_UUID}" name="textField">
    <table_ref uuid="{TABLE_UUID}" name="TableName"/>
  </field_ref>
</index>
```

**Composite index:**
```xml
<index kind="regular" unique_keys="false" name="IndexName" uuid="{INDEX_UUID}" type="1">
  <field_ref uuid="{FIELD1_UUID}" name="field1">
    <table_ref uuid="{TABLE_UUID}" name="TableName"/>
  </field_ref>
  <field_ref uuid="{FIELD2_UUID}" name="field2">
    <table_ref uuid="{TABLE_UUID}" name="TableName"/>
  </field_ref>
</index>
```

### Relations

Relations link tables via foreign key → primary key. They enable ORDA navigation (the "R" in ORDA).

Reference: https://developer.4d.com/docs/ORDA/dsmapping

**In ORDA:**
- `name_Nto1` → creates a `relatedEntity` attribute (e.g., `person.employer` returns one Company)
- `name_1toN` → creates a `relatedEntities` attribute (e.g., `company.employees` returns many Persons)

```xml
<relation name_Nto1="employer" name_1toN="employees" uuid="{REL_UUID}"
  auto_load_Nto1="false" auto_load_1toN="false"
  integrity="reject" state="1">
  <related_field kind="source">
    <field_ref uuid="{FK_FIELD_UUID}" name="companyID">
      <table_ref uuid="{FK_TABLE_UUID}" name="Persons"/>
    </field_ref>
  </related_field>
  <related_field kind="destination">
    <field_ref uuid="{PK_FIELD_UUID}" name="ID">
      <table_ref uuid="{PK_TABLE_UUID}" name="Companies"/>
    </field_ref>
  </related_field>
</relation>
```

**Attributes:**
- `name_Nto1`: Relation name from Many→One (foreign key side → primary key side)
- `name_1toN`: Relation name from One→Many (primary key side → foreign key side)
- `auto_load_Nto1="false"`, `auto_load_1toN="false"`: Always `false` for ORDA/modern usage
- `integrity`: `"none"` (allow orphans), `"reject"` (block delete if related records exist), `"delete"` (cascade delete)
- `state="1"`: Active relation
- `foreign_key="false"`: Use `false` for 4D relations (use `true` for SQL-style foreign key constraints)

**Important:**
- The foreign key field must be the **same type** as the primary key it references
- Always create an **index on the foreign key field** (Cluster B-tree `type="3"` is ideal)
- `kind="source"` = Many side (foreign key), `kind="destination"` = One side (primary key)

### Comments on Tables and Fields

**Table comment** (inside `<table_extra>`):
```xml
<table_extra>
  <comment format="text">Description of this table</comment>
</table_extra>
```

**Field comment** (inside `<field_extra>`):
```xml
<field_extra>
  <comment format="text">Description of this field</comment>
</field_extra>
```

For agent-generated projects, use only `format="text"` (skip RTF).

## Catalog Element Ordering

The catalog XML must follow this order within `<base>`:
1. `<schema name="DEFAULT_SCHEMA"/>`
2. `<table>` elements (each containing fields, primary_key, optional table_extra)
3. `<relation>` elements
4. `<index>` elements
5. `<base_extra>` (journal settings)

## Methods (.4dm files)

Methods are plain text files in `Project/Sources/Methods/`. When `tokenizedText` is `false`, write **plain 4D code without token suffixes** (no `:C###` or `:K##:##`).

**File**: `Project/Sources/Methods/{MethodName}.4dm`

- Method names follow the same identifier rules (max 31 chars, no reserved names).
- Methods are auto-discovered by 4D — no registration needed.
- For validation/testing, see `tool4d-cli.md`.

## Complete Example

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE base SYSTEM "http://www.4d.com/dtd/2007/base.dtd" >
<base name="MyApp" uuid="B0000000000000000000000000000000" collation_locale="en-gb">
  <schema name="DEFAULT_SCHEMA"/>

  <table name="Companies" uuid="A0010000000000000000000000000000" id="1">
    <field name="ID" uuid="F0010001000000000000000000000000" type="4" unique="true" autosequence="true" not_null="true" id="1"/>
    <field name="companyName" uuid="F0010002000000000000000000000000" type="10" limiting_length="255" id="2"/>
    <field name="address" uuid="F0010003000000000000000000000000" type="10" limiting_length="255" id="3"/>
    <primary_key field_name="ID" field_uuid="F0010001000000000000000000000000"/>
  </table>

  <table name="Employees" uuid="A0020000000000000000000000000000" id="2">
    <field name="ID" uuid="F0020001000000000000000000000000" type="4" unique="true" autosequence="true" not_null="true" id="1"/>
    <field name="firstName" uuid="F0020002000000000000000000000000" type="10" limiting_length="80" id="2"/>
    <field name="lastName" uuid="F0020003000000000000000000000000" type="10" limiting_length="80" id="3"/>
    <field name="companyID" uuid="F0020004000000000000000000000000" type="4" id="4"/>
    <primary_key field_name="ID" field_uuid="F0020001000000000000000000000000"/>
  </table>

  <relation name_Nto1="employer" name_1toN="employees" uuid="D0010000000000000000000000000000"
    auto_load_Nto1="false" auto_load_1toN="false" integrity="reject" state="1">
    <related_field kind="source">
      <field_ref uuid="F0020004000000000000000000000000" name="companyID">
        <table_ref uuid="A0020000000000000000000000000000" name="Employees"/>
      </field_ref>
    </related_field>
    <related_field kind="destination">
      <field_ref uuid="F0010001000000000000000000000000" name="ID">
        <table_ref uuid="A0010000000000000000000000000000" name="Companies"/>
      </field_ref>
    </related_field>
  </relation>

  <index kind="regular" unique_keys="true" uuid="C0010001000000000000000000000000" type="1">
    <field_ref uuid="F0010001000000000000000000000000" name="ID">
      <table_ref uuid="A0010000000000000000000000000000" name="Companies"/>
    </field_ref>
  </index>
  <index kind="regular" unique_keys="true" uuid="C0020001000000000000000000000000" type="1">
    <field_ref uuid="F0020001000000000000000000000000" name="ID">
      <table_ref uuid="A0020000000000000000000000000000" name="Employees"/>
    </field_ref>
  </index>
  <index kind="regular" unique_keys="false" uuid="C0020004000000000000000000000000" type="3">
    <field_ref uuid="F0020004000000000000000000000000" name="companyID">
      <table_ref uuid="A0020000000000000000000000000000" name="Employees"/>
    </field_ref>
  </index>

  <base_extra>
    <journal_file journal_file_enabled="true"/>
  </base_extra>
</base>
```
