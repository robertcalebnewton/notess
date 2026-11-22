# NvChad Usage Manual (Practical Guide)

This manual focuses on **everyday NvChad usage**: moving around, buffers, file tree, Treesitter, LSP, and common tools.  
Keybindings are based on **default NvChad settings**.

> **Leader key**: `Space`

---

## 1. Basic Movement (Vim Essentials)

### Normal Mode Movement
| Key        | Action                   |
| ---------- | ------------------------ |
| `h j k l`  | Left / Down / Up / Right |
| `w`        | Next word                |
| `b`        | Previous word            |
| `0`        | Start of line            |
| `$`        | End of line              |
| `gg`       | Top of file              |
| `G`        | Bottom of file           |
| `Ctrl + d` | Page down                |
| `Ctrl + u` | Page up                  |

### Insert / Normal Mode
| Key   | Action               |
| ----- | -------------------- |
| `i`   | Insert before cursor |
| `a`   | Insert after cursor  |
| `o`   | New line below       |
| `Esc` | Back to Normal mode  |

---

## 2. Buffers (Open Files)

Buffers are open files in memory.

### Buffer Navigation
| Key | Action |
|----|-------|
| `Tab` | Next buffer |
| `Shift + Tab` | Previous buffer |
| `Space + x` | Close current buffer |

### Pick Buffers
| Key | Action |
|----|-------|
| `Space + b` | Buffer picker (choose by letter) |

---

## 3. File Tree (NvimTree)

### Toggle File Explorer
| Key | Action |
|----|-------|
| `Ctrl + n` | Toggle file tree |

### Inside File Tree
| Key | Action |
|----|-------|
| `Enter` | Open file / folder |
| `a` | Create file |
| `d` | Delete file |
| `r` | Rename |
| `m` | Move file |
| `H` | Toggle hidden files |

---

## 4. File & Text Search (Telescope)

### Find Files
| Key | Action |
|----|-------|
| `Space + f + f` | Find files |
| `Space + f + o` | Recently opened files |

### Live Search
| Key | Action |
|----|-------|
| `Space + f + w` | Search word in project |
| `Space + /` | Search in current buffer |

### Navigation in Telescope
| Key | Action |
|----|-------|
| `Ctrl + j / Ctrl + k` | Move selection |
| `Enter` | Open selection |
| `Esc` | Close |

---

## 5. Treesitter (Syntax & Selection)

Treesitter provides better syntax highlighting and smart selection.

### Incremental Selection
| Key | Action |
|----|-------|
| `Ctrl + n` | Expand selection |
| `Ctrl + m` | Shrink selection |

### Syntax Highlighting
- Enabled automatically
- Works per language (JS, TS, Python, Lua, etc.)

---

## 6. LSP (Language Server Protocol)

### Go To & Navigation
| Key | Action |
|----|-------|
| `gd` | Go to definition |
| `gr` | Find references |
| `gi` | Go to implementation |
| `K` | Hover documentation |

### Code Actions & Rename
| Key | Action |
|----|-------|
| `Space + l + a` | Code actions |
| `Space + l + r` | Rename symbol |

### Diagnostics (Errors & Warnings)
| Key | Action |
|----|-------|
| `Space + l + d` | Show diagnostics |
| `]d` | Next diagnostic |
| `[d` | Previous diagnostic |

---

## 7. Formatting & Indentation

### Format File
| Key | Action |
|----|-------|
| `Space + l + f` | Format current file |

### Indentation
| Key | Action |
|----|-------|
| `>>` | Indent right |
| `<<` | Indent left |
| `=` | Auto-indent |

---

## 8. Commenting Code

| Mode | Key | Action |
|----|----|------|
| Normal | `gcc` | Toggle line comment |
| Visual | `gc` | Comment selection |

---

## 9. Terminal

### Open Terminal
| Key | Action |
|----|-------|
| `Space + h + t` | Horizontal terminal |
| `Space + v + t` | Vertical terminal |
| `Alt + i` | Floating terminal |

### Terminal Mode
| Key | Action |
|----|-------|
| `Esc` | Normal mode |
| `Ctrl + x` | Close terminal |

---

## 10. Window Management

### Split Windows
| Key | Action |
|----|-------|
| `Space + v` | Vertical split |
| `Space + h` | Horizontal split |

### Move Between Windows
| Key | Action |
|----|-------|
| `Ctrl + h/j/k/l` | Move focus |

---

## 11. Git (Gitsigns)

### Git Actions
| Key | Action |
|----|-------|
| `Space + g + b` | Blame line |
| `Space + g + h` | Preview hunk |
| `]c` | Next git change |
| `[c` | Previous git change |

---

## 12. Useful Commands

| Command | Description |
|------|-------------|
| `:Lazy` | Plugin manager |
| `:Mason` | LSP/DAP installer |
| `:checkhealth` | Debug config |
| `:qa` | Quit all |

---

## 13. Learning Tips

- Use `:WhichKey` or wait after pressing `Space`
- Learn **Normal mode first**
- Don’t memorize everything at once
- Use Telescope for *everything*

---

## 14. Useful Resources

- NvChad Docs: https://nvchad.com
- NvChad GitHub: https://github.com/NvChad/NvChad
- Neovim Help: `:help`

---

Happy hacking 🚀  
NvChad is fast once muscle memory kicks in.

