# Git Status: Parsing & Presentation

## 1. Parsing (`--porcelain=v2`)

### Headers
- `# branch.oid <hash>`: Commit SHA.
- `# branch.head <name>`: Branch name.
- `# branch.upstream <remote>/<branch>`: Tracking branch.
- `# branch.ab +<ahead> -<behind>`: Sync status.

### File Status (`XY <path>`)
`X`=Index, `Y`=WorkTree.

| XY | Staged | Unstaged | Meaning |
|:--:|:--:|:--:|:--|
| `M.` | ✓ | | Modified (Staged) |
| `.M` | | ✓ | Modified (Unstaged) |
| `MM` | ✓ | ✓ | Modified (Both) |
| `A.` | ✓ | | Added |
| `AM` | ✓ | ✓ | Added & Modified |
| `D.` | ✓ | | Deleted (Staged) |
| `.D` | | ✓ | Deleted (Unstaged) |
| `R.` | ✓ | | Renamed |
| `??` | | | Untracked |
| `!!` | | | Ignored |

## 2. Presentation Rules

- **Structure:** `📍 Header` -> `📝 Changes` -> `✨ Clean`.
- **Separators:** `═` (x60) top/bottom only.
- **Tree:** `├─`, `└─`, `│`. Sort files alphabetically.
- **Truncation:** >10 files? Show 8 + "...and N more".

### Emojis & Styles
| Emoji | Meaning | Context |
|:--:|:--|:--|
| `📍` | Branch Info | Header |
| `✨` | Clean | Body |
| `✓` | Staged/Sync | Green |
| `⚡` | Unstaged | Yellow |
| `•` | Untracked | Yellow |
| `↑/↓` | Ahead/Behind | Yellow/Red |

### Templates

**Clean:**
```
════════════════════════════════════════════════════════════
📍 Branch: <name>
   └─ Upstream: <remote> <emoji> <status>

✨ Working tree clean
════════════════════════════════════════════════════════════
```

**Dirty:**
```
════════════════════════════════════════════════════════════
📍 Branch: <name>
   ├─ Upstream: <remote> <emoji>
   └─ Status: <desc>

📝 Working Tree Changes

[✓ Staged (<n>)]
  ├─ Modified: <files>
  └─ Added: <files>

[⚡ Unstaged (<n>)]
  └─ Modified: <files>

[• Untracked (<n>)]
  └─ <files>
════════════════════════════════════════════════════════════
```
