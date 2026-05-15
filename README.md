# project documentation... documentation 

A repeatable template for organizing project work — designed to be simple for humans to navigate and clear enough for AI tools to find files without being given explicit paths.

---

## The Problem

The original structure grouped files by type within each project:

```
projects/active/project_alpha/sql/initial_query.sql
projects/active/project_alpha/md/requirements.md
projects/active/project_alpha/csv/source_data_sample.csv
```

This created two problems:
1. **Three levels of navigation** to reach any file — project → file type → file
2. **File type is already in the extension** — the `sql/` folder adds no information

---

## The Solution

Flatten to two levels: status → project → files. File type is derivable from the extension; it does not need to be a folder.

```
projects/active/project_gamma/initial_query.sql
projects/active/project_gamma/requirements.md
projects/active/project_gamma/source_data_sample.csv
```

---

## Structure

```
projects/
├── active/
│   └── project_name/
│       ├── J-XXXXX_project_name.md
│       └── (all project files, flat)
├── complete/
│   └── project_name/
│       ├── J-XXXXX_project_name.md
│       └── (all project files, flat)
└── paused/
    └── project_name/
        ├── J-XXXXX_project_name.md
        └── (all project files, flat)
```

### Status Folders

| Folder | Meaning |
|---|---|
| `active/` | Work in progress |
| `complete/` | Delivered and closed |
| `paused/` | Started but fully deprioritized — not abandoned, not active |

### Rules

- **No subfolders within a project** — all files sit directly in the project folder
- **Moving between states** means moving the folder between `active/`, `complete/`, and `paused/` — the folder name never changes
- **Every project** gets a `p-XXXXX_project_name.md` file — both the ticket number and project name are visible in the file listing without opening anything
- **Power BI projects** store `.pbip` folders as `pbip_[filename]` where `[filename]` is the `.pbip` filename without the extension — makes the report identity visible in the directory listing and supports multiple `.pbip` projects within a single ticket folder
- **File naming** should be descriptive enough to identify purpose without opening the file

---

## `J-XXXXX_project_name.md`

Each project contains a `J-XXXXX_project_name.md` as a lightweight anchor back to the source of truth in your ticketing system. It is not a documentation file — it is a pointer.

The ticket number and project name in the filename make both visible in any file listing without opening anything. It is always the first thing you see when you open a project folder.

```markdown
# project_name

project_identifier: [J-12345](https://your-ticketing-system/J-12345)
```

This gives AI tools a guaranteed, consistent entry point for every project — reading this file is always sufficient to identify what a project is and where to find its full context.

---

## Why It Works for AI

A flat, predictable path means an AI assistant can:
- List all files in a project with a single directory read
- Identify file types from extensions without traversing subfolders
- Locate any project by scanning `active/`, `complete/`, or `paused/` without needing an explicit path in the prompt
- Read `J-XXXXX_project_name.md` to identify the project and link to its ticket without asking the user

---

## Reference Projects

`project_alpha` and `project_beta` are preserved to illustrate the **old structure** (file-type subfolders). All projects from `gamma` onward use the flat structure.

| Project | Status | Ticket | Structure |
|---|---|---|---|
| project_alpha | active | J-10001 | old (type subfolders) |
| project_beta | complete | J-10002 | old (type subfolders) |
| project_gamma | paused | J-10003 | new (flat) |
| project_delta | paused | J-10004 | new (flat) |
| project_epsilon | paused | J-10005 | new (flat) |
| project_zeta | paused | J-10006 | new (flat) |
| project_eta | paused | J-10007 | new (flat) |
| project_theta | paused | J-10008 | new (flat) |
| project_iota | complete | J-10009 | new (flat) |
| project_kappa | complete | J-10010 | new (flat) |
| project_lambda | complete | J-10011 | new (flat) |
| project_mu | complete | J-10012 | new (flat) |
| project_nu | complete | J-10013 | new (flat) |
