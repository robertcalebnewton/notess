# Building TUI Apps with Textual

> **Textual** is the most powerful Python TUI framework available. It brings a CSS-like styling system, reactive state, async-first architecture, and a rich widget library — all running in the terminal. This tutorial takes you from zero to a fully functional multi-panel TUI app.

---

## Table of Contents

1. [Why Textual](#1-why-textual)
2. [Installation](#2-installation)
3. [Project Structure](#3-project-structure)
4. [Your First App](#4-your-first-app)
5. [Widgets](#5-widgets)
6. [Layouts with CSS](#6-layouts-with-css)
7. [Reactive State](#7-reactive-state)
8. [Events & Messages](#8-events--messages)
9. [Actions & Keybindings](#9-actions--keybindings)
10. [Screens & Navigation](#10-screens--navigation)
11. [Workers (Async Tasks)](#11-workers-async-tasks)
12. [Full Project: Task Manager TUI](#12-full-project-task-manager-tui)
13. [Debugging & Dev Tools](#13-debugging--dev-tools)
14. [Useful Patterns](#14-useful-patterns)

---

## 1. Why Textual

| Feature | Textual | Curses | Rich | Urwid |
|---|---|---|---|---|
| CSS-like layout | ✅ | ❌ | ❌ | ❌ |
| Reactive state | ✅ | ❌ | ❌ | ❌ |
| Async-first | ✅ | ❌ | ❌ | Partial |
| Mouse support | ✅ | Partial | ❌ | Partial |
| Built-in widgets | ✅ Rich | Minimal | ❌ | Limited |
| Devtools | ✅ | ❌ | ❌ | ❌ |

Textual is built on top of [Rich](https://github.com/Textualize/rich) by the same team (Textualize), so Rich renderables work natively inside Textual widgets.

---

## 2. Installation

```bash
pip install textual

# Install dev tools for the CSS inspector (highly recommended)
pip install textual-dev
```

Verify:

```bash
python -m textual
```

You should see the Textual demo app in your terminal.

---

## 3. Project Structure

A typical Textual project:

```
my_tui/
├── app.py          # Main App class, entry point
├── screens/
│   ├── main.py     # Main screen
│   └── settings.py # Settings screen
├── widgets/
│   └── sidebar.py  # Custom widgets
└── styles/
    └── app.tcss    # Textual CSS (TCSS)
```

You can inline CSS inside Python strings, or load `.tcss` files — external files are recommended for anything non-trivial.

---

## 4. Your First App

```python
# app.py
from textual.app import App, ComposeResult
from textual.widgets import Header, Footer, Label

class HelloApp(App):
    """A minimal Textual app."""

    CSS = """
    Label {
        content-align: center middle;
        height: 100%;
        color: $accent;
        text-style: bold;
    }
    """

    def compose(self) -> ComposeResult:
        yield Header()
        yield Label("Hello, Terminal!")
        yield Footer()

if __name__ == "__main__":
    HelloApp().run()
```

Run it:

```bash
python app.py
```

**Key concepts shown here:**

- `App` is your root class. Everything lives inside it.
- `compose()` is where you build the widget tree by `yield`-ing widgets.
- `CSS` is an inline class variable for styles (use `.tcss` files for larger projects).
- `Header` and `Footer` are built-in widgets — Footer shows keybindings automatically.

---

## 5. Widgets

Textual ships with a rich set of built-in widgets.

### Built-in Widgets Quick Reference

```python
from textual.widgets import (
    Header, Footer,         # chrome
    Label, Static,          # text display
    Button,                 # clickable button
    Input,                  # text input field
    TextArea,               # multiline editor
    ListView, ListItem,     # scrollable list
    DataTable,              # grid/table
    Tree,                   # collapsible tree
    Log,                    # append-only log output
    ProgressBar,            # progress indicator
    Switch, Checkbox,       # toggles
    Select,                 # dropdown
    Tabs, Tab,              # tab switcher
    Markdown,               # render markdown
    Sparkline,              # inline sparkline chart
    RichLog,                # Rich-powered log
    TabbedContent,          # tabs with content panels
    DirectoryTree,          # filesystem browser
)
```

### Creating a Custom Widget

```python
from textual.app import App, ComposeResult
from textual.widget import Widget
from textual.reactive import reactive

class Counter(Widget):
    """A custom widget that displays a count."""

    count: reactive[int] = reactive(0)

    def render(self) -> str:
        return f"Count: [bold cyan]{self.count}[/bold cyan]"

    def increment(self) -> None:
        self.count += 1
```

- Override `render()` for simple widgets that return a string or Rich renderable.
- Override `compose()` to build composite widgets from other widgets.
- Use `reactive` for state that automatically re-renders when changed.

---

## 6. Layouts with CSS

Textual uses a subset of CSS called **TCSS** (Textual CSS). Layout is primarily done with `layout: horizontal` / `layout: vertical` and `grid`.

### Vertical Layout (default)

```tcss
/* styles/app.tcss */
Screen {
    layout: vertical;
}
```

### Horizontal Split (sidebar + main)

```python
from textual.app import App, ComposeResult
from textual.widgets import Header, Footer, Label
from textual.containers import Horizontal, Vertical

class LayoutApp(App):

    CSS = """
    #sidebar {
        width: 25%;
        background: $panel;
        border-right: solid $accent;
        padding: 1 2;
    }
    #main {
        width: 75%;
        padding: 1 2;
    }
    """

    def compose(self) -> ComposeResult:
        yield Header()
        with Horizontal():
            with Vertical(id="sidebar"):
                yield Label("Navigation")
            with Vertical(id="main"):
                yield Label("Main Content")
        yield Footer()
```

### Grid Layout

```tcss
#grid {
    layout: grid;
    grid-size: 3 2;        /* 3 columns, 2 rows */
    grid-gutter: 1;
}

.cell {
    height: 100%;
    border: solid $accent;
}
```

### Useful CSS Properties

```tcss
/* Sizing */
width: 50%;          /* percentage */
width: 30;           /* fixed columns */
height: 1fr;         /* fractional unit — takes remaining space */
min-width: 20;
max-height: 10;

/* Spacing */
padding: 1 2;        /* top/bottom left/right */
margin: 1;

/* Display */
display: block;      /* show */
display: none;       /* hide */
visibility: hidden;  /* invisible but takes space */

/* Color — use Textual design tokens */
background: $surface;
color: $text;
border: solid $accent;

/* Text */
text-style: bold italic;
text-align: center;
content-align: center middle;  /* for Label/Static */
```

### Textual Design Tokens

| Token | Meaning |
|---|---|
| `$background` | App background |
| `$surface` | Widget surface |
| `$panel` | Panel/sidebar background |
| `$accent` | Highlight color |
| `$text` | Primary text |
| `$text-muted` | Dimmed text |
| `$success` | Green |
| `$warning` | Yellow |
| `$error` | Red |

---

## 7. Reactive State

Reactive variables are the core of Textual's data flow. When a reactive changes, Textual automatically re-renders the widget and calls any watchers.

```python
from textual.app import App, ComposeResult
from textual.reactive import reactive
from textual.widgets import Label, Button
from textual.containers import Vertical

class ReactiveApp(App):

    # Reactive declared at class level
    score: reactive[int] = reactive(0)

    def compose(self) -> ComposeResult:
        yield Label(id="score-label")
        yield Button("Add Point", id="add")
        yield Button("Reset", id="reset")

    # Watcher: called automatically when `score` changes
    def watch_score(self, new_score: int) -> None:
        self.query_one("#score-label", Label).update(
            f"Score: [bold green]{new_score}[/bold green]"
        )

    def on_button_pressed(self, event: Button.Pressed) -> None:
        if event.button.id == "add":
            self.score += 1
        elif event.button.id == "reset":
            self.score = 0
```

### Reactive Options

```python
# Basic reactive
name: reactive[str] = reactive("default")

# Recomposes the widget on change (calls compose() again)
name: reactive[str] = reactive("default", recompose=True)

# Init=False: watcher not called on startup
name: reactive[str] = reactive("default", init=False)
```

---

## 8. Events & Messages

Textual uses a message-passing system. Widgets post messages up the DOM tree; parent widgets handle them with `on_*` methods.

### Handling Built-in Events

```python
from textual.widgets import Button, Input

class MyApp(App):

    def on_button_pressed(self, event: Button.Pressed) -> None:
        """Called when ANY button in the app is pressed."""
        print(event.button.id)

    def on_input_submitted(self, event: Input.Submitted) -> None:
        """Called when the user presses Enter in an Input."""
        print(event.value)

    def on_input_changed(self, event: Input.Changed) -> None:
        """Called on every keystroke in an Input."""
        print(event.value)
```

### Custom Messages

Define custom messages to communicate between widgets and their parents:

```python
from textual.message import Message
from textual.widget import Widget
from textual.app import App

class SearchBox(Widget):

    class Searched(Message):
        """Posted when the user submits a search."""
        def __init__(self, query: str) -> None:
            super().__init__()
            self.query = query

    def on_input_submitted(self, event) -> None:
        # Post the message up the DOM
        self.post_message(self.Searched(event.value))


class MyApp(App):

    def on_search_box_searched(self, event: SearchBox.Searched) -> None:
        # Handler name format: on_{widget_class}_{message_class} (snake_case)
        self.notify(f"Searching for: {event.query}")
```

### Using `@on` Decorator

```python
from textual.on import on
from textual.widgets import Button

class MyApp(App):

    @on(Button.Pressed, "#submit")   # only handle the #submit button
    def handle_submit(self, event: Button.Pressed) -> None:
        self.notify("Submitted!")

    @on(Button.Pressed, "#cancel")
    def handle_cancel(self, event: Button.Pressed) -> None:
        self.notify("Cancelled.")
```

---

## 9. Actions & Keybindings

Actions are named methods on your App or Screen that can be triggered by keybindings or buttons.

```python
from textual.app import App, ComposeResult
from textual.widgets import Header, Footer, Label
from textual.binding import Binding

class KeyApp(App):

    BINDINGS = [
        Binding("q", "quit", "Quit"),
        Binding("d", "toggle_dark", "Toggle Dark Mode"),
        Binding("ctrl+n", "new_item", "New Item"),
    ]

    def compose(self) -> ComposeResult:
        yield Header()
        yield Label("Press keys to trigger actions")
        yield Footer()   # Footer auto-displays BINDINGS

    def action_toggle_dark(self) -> None:
        """Action methods are prefixed with action_"""
        self.dark = not self.dark

    def action_new_item(self) -> None:
        self.notify("Creating new item...")
```

Action methods are always named `action_{action_name}`. The Footer widget reads `BINDINGS` automatically and displays them.

---

## 10. Screens & Navigation

Screens let you build multi-page TUI apps with a navigation stack.

```python
from textual.app import App, ComposeResult
from textual.screen import Screen, ModalScreen
from textual.widgets import Header, Footer, Button, Label
from textual.binding import Binding

class SettingsScreen(Screen):
    """A full screen pushed onto the stack."""

    BINDINGS = [Binding("escape", "app.pop_screen", "Back")]

    def compose(self) -> ComposeResult:
        yield Header()
        yield Label("Settings Screen")
        yield Button("Back", id="back")
        yield Footer()

    def on_button_pressed(self, event: Button.Pressed) -> None:
        self.app.pop_screen()


class ConfirmModal(ModalScreen[bool]):
    """A modal dialog that returns a value."""

    def compose(self) -> ComposeResult:
        yield Label("Are you sure?")
        yield Button("Yes", id="yes", variant="error")
        yield Button("No", id="no")

    def on_button_pressed(self, event: Button.Pressed) -> None:
        self.dismiss(event.button.id == "yes")  # returns True or False


class MainApp(App):

    SCREENS = {
        "settings": SettingsScreen,  # register named screens
    }

    def compose(self) -> ComposeResult:
        yield Header()
        yield Button("Open Settings", id="settings")
        yield Button("Delete (confirm)", id="delete")
        yield Footer()

    def on_button_pressed(self, event: Button.Pressed) -> None:
        if event.button.id == "settings":
            self.push_screen("settings")       # push named screen
        elif event.button.id == "delete":
            self.push_screen(ConfirmModal(), self.handle_confirm)

    def handle_confirm(self, confirmed: bool) -> None:
        """Callback from ConfirmModal."""
        if confirmed:
            self.notify("Deleted!", severity="warning")
```

**Screen navigation methods:**

```python
self.app.push_screen(screen)          # push a new screen
self.app.push_screen(screen, callback) # push with result callback
self.app.pop_screen()                 # go back
self.app.switch_screen(screen)        # replace current screen
self.dismiss(value)                   # return value from modal
```

---

## 11. Workers (Async Tasks)

Use workers to run blocking or long-running tasks without freezing the UI.

```python
from textual.app import App, ComposeResult
from textual.widgets import Button, Log
from textual.worker import Worker, get_current_worker
import asyncio

class WorkerApp(App):

    def compose(self) -> ComposeResult:
        yield Button("Start Task", id="start")
        yield Log(id="log")

    def on_button_pressed(self, event: Button.Pressed) -> None:
        self.run_worker(self.do_work(), exclusive=True)

    async def do_work(self) -> None:
        log = self.query_one(Log)
        worker = get_current_worker()
        for i in range(10):
            if worker.is_cancelled:
                break
            log.write_line(f"Step {i + 1}/10...")
            await asyncio.sleep(0.5)
        log.write_line("Done!")

    def on_worker_state_changed(self, event: Worker.StateChanged) -> None:
        """Monitor worker state changes."""
        self.log(event)
```

For CPU-bound blocking code, use `thread=True`:

```python
# Runs in a thread pool — safe for blocking I/O or CPU work
self.run_worker(self.blocking_task, thread=True)
```

---

## 12. Full Project: Task Manager TUI

A complete, runnable task manager with add/complete/delete functionality.

```python
# task_manager.py
from textual.app import App, ComposeResult
from textual.binding import Binding
from textual.containers import Horizontal, Vertical, ScrollableContainer
from textual.message import Message
from textual.reactive import reactive
from textual.widget import Widget
from textual.widgets import (
    Button, Footer, Header, Input, Label, Static
)
from dataclasses import dataclass, field
import uuid


@dataclass
class Task:
    id: str = field(default_factory=lambda: str(uuid.uuid4())[:8])
    title: str = ""
    done: bool = False


class TaskItem(Widget):
    """A single task row."""

    class Toggled(Message):
        def __init__(self, task_id: str) -> None:
            super().__init__()
            self.task_id = task_id

    class Deleted(Message):
        def __init__(self, task_id: str) -> None:
            super().__init__()
            self.task_id = task_id

    def __init__(self, task: Task) -> None:
        super().__init__()
        self.task = task

    def compose(self) -> ComposeResult:
        status = "✓" if self.task.done else "○"
        style = "strike dim" if self.task.done else ""
        with Horizontal(classes="task-row"):
            yield Button(status, id="toggle", classes="toggle-btn")
            yield Label(
                f"[{style}]{self.task.title}[/{style}]" if style
                else self.task.title,
                classes="task-label"
            )
            yield Button("✕", id="delete", classes="delete-btn")

    def on_button_pressed(self, event: Button.Pressed) -> None:
        event.stop()
        if event.button.id == "toggle":
            self.post_message(self.Toggled(self.task.id))
        elif event.button.id == "delete":
            self.post_message(self.Deleted(self.task.id))


class TaskManagerApp(App):

    CSS = """
    Screen {
        layout: vertical;
    }

    #input-bar {
        height: 3;
        layout: horizontal;
        padding: 0 1;
        background: $panel;
        border-bottom: solid $accent;
    }

    #task-input {
        width: 1fr;
    }

    #add-btn {
        width: 10;
        margin-left: 1;
    }

    #task-list {
        height: 1fr;
        padding: 1 2;
    }

    #stats {
        height: 3;
        content-align: center middle;
        background: $panel;
        border-top: solid $accent;
        color: $text-muted;
    }

    .task-row {
        height: 3;
        layout: horizontal;
        margin-bottom: 1;
        background: $surface;
        border: solid $panel;
    }

    .task-row:hover {
        border: solid $accent;
    }

    .toggle-btn {
        width: 5;
    }

    .task-label {
        width: 1fr;
        content-align: left middle;
        padding: 0 1;
    }

    .delete-btn {
        width: 5;
        color: $error;
    }
    """

    BINDINGS = [
        Binding("ctrl+n", "focus_input", "New Task"),
        Binding("ctrl+a", "complete_all", "Complete All"),
        Binding("ctrl+x", "clear_done", "Clear Done"),
        Binding("q", "quit", "Quit"),
    ]

    tasks: reactive[list[Task]] = reactive(list, recompose=True)

    def compose(self) -> ComposeResult:
        yield Header(show_clock=True)
        with Horizontal(id="input-bar"):
            yield Input(placeholder="New task...", id="task-input")
            yield Button("Add  ↵", id="add-btn", variant="primary")
        with ScrollableContainer(id="task-list"):
            if not self.tasks:
                yield Static(
                    "[dim]No tasks yet — press Ctrl+N to add one[/dim]",
                    id="empty-msg"
                )
            for task in self.tasks:
                yield TaskItem(task)
        done = sum(1 for t in self.tasks if t.done)
        yield Static(
            f"{done}/{len(self.tasks)} completed",
            id="stats"
        )
        yield Footer()

    def action_focus_input(self) -> None:
        self.query_one("#task-input", Input).focus()

    def action_complete_all(self) -> None:
        for task in self.tasks:
            task.done = True
        self.mutate_reactive(TaskManagerApp.tasks)

    def action_clear_done(self) -> None:
        self.tasks = [t for t in self.tasks if not t.done]

    def on_button_pressed(self, event: Button.Pressed) -> None:
        if event.button.id == "add-btn":
            self._add_task()

    def on_input_submitted(self, event: Input.Submitted) -> None:
        self._add_task()

    def _add_task(self) -> None:
        input_widget = self.query_one("#task-input", Input)
        title = input_widget.value.strip()
        if title:
            self.tasks = self.tasks + [Task(title=title)]
            input_widget.value = ""
            self.notify(f"Added: {title}")

    def on_task_item_toggled(self, event: TaskItem.Toggled) -> None:
        for task in self.tasks:
            if task.id == event.task_id:
                task.done = not task.done
        self.mutate_reactive(TaskManagerApp.tasks)

    def on_task_item_deleted(self, event: TaskItem.Deleted) -> None:
        self.tasks = [t for t in self.tasks if t.id != event.task_id]
        self.notify("Task removed", severity="warning")


if __name__ == "__main__":
    TaskManagerApp().run()
```

Run it:

```bash
python task_manager.py
```

**What this demo covers:**

- Custom widget with `compose()` and custom `Message` subclasses
- `reactive` list with `recompose=True` driving full re-renders
- `mutate_reactive()` for mutating mutable reactive objects in place
- `BINDINGS` with named actions
- `ScrollableContainer` for scrollable regions
- `notify()` for toast notifications
- Full TCSS styling with design tokens and hover states

---

## 13. Debugging & Dev Tools

### Textual Devtools (CSS Inspector)

```bash
# In one terminal, start the devtools server
textual devtools

# In another terminal, run your app with --dev
python app.py --dev
```

Open `http://localhost:8081` in your browser for a live CSS inspector, DOM tree, and event log.

### Logging

Textual redirects `print()` to the devtools console, not stdout (which would break the TUI). Use:

```python
# Inside any App/Widget/Screen method:
self.log("debug message")
self.log.warning("something odd")
self.log.error("something broke")

# Or use the standard logging module — it also routes to devtools
import logging
logging.getLogger(__name__).info("hello")
```

### Screenshotting

```python
# Save an SVG screenshot of the current state
app.save_screenshot("screenshot.svg")
```

Or run:

```bash
textual run --screenshot app.py
```

---

## 14. Useful Patterns

### Querying the DOM

```python
# Single widget by type
label = self.query_one(Label)

# Single widget by CSS selector
label = self.query_one("#my-label", Label)

# Multiple widgets
buttons = self.query(Button)
for btn in buttons:
    btn.disabled = True

# Query from within a widget (scoped to subtree)
item = self.query_one(".task-row Button")
```

### Show / Hide Widgets

```python
widget.display = False    # hides + removes from layout
widget.visible = False    # invisible but keeps space
widget.add_class("hidden")    # use CSS: .hidden { display: none; }
widget.remove_class("hidden")
widget.toggle_class("active")
```

### Notifications (Toast Messages)

```python
self.notify("Task saved!")
self.notify("File not found", severity="warning")
self.notify("Permission denied", severity="error", timeout=8)
```

### Updating Widget Content

```python
# Label / Static
self.query_one("#status", Label).update("New text")

# Log widget
log = self.query_one(Log)
log.write_line("new line")
log.clear()

# DataTable
table = self.query_one(DataTable)
table.add_column("Name")
table.add_row("Alice", "Bob")
```

### Running an App Headlessly (Testing)

```python
from textual.testing import AppTest

async def test_task_app():
    async with TaskManagerApp().run_test() as pilot:
        await pilot.click("#add-btn")
        await pilot.press("ctrl+n")
```

---

## Further Reading

- [Official Textual Docs](https://textual.textualize.io/)
- [Textual GitHub](https://github.com/Textualize/textual) — examples folder is excellent
- [Textualize Discord](https://discord.gg/Enf6Z3qhVr) — active, responsive community
- [textual-dev PyPI](https://pypi.org/project/textual-dev/) — devtools
- [Rich Docs](https://rich.readthedocs.io/) — Rich renderables work inside Textual widgets
