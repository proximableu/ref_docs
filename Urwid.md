Here is a complete, high-density, context-ready **Urwid LLM Reference Sheet**. It is designed to be pasted directly into your LLM chat context or custom system prompt to eliminate core API hallucinations, layout dimension errors, and event-bubbling bugs.

---

# Copy-Paste System Reference: Urwid TUI Framework

You are writing Python code using the **Urwid** terminal console user interface library. Adhere strictly to the following framework constraints, layout rules, and API specifications.

---

## 1. Core Paradigm: The Three Sizing Archetypes

Urwid widgets have strict dimension contracts. Passing the wrong archetype into a container without a constraint layout wrapper causes **`ValueError: Not a ... widget`** runtime crashes.

| Sizing Type | Core API Method signature | Sizing Contract | Examples |
| --- | --- | --- | --- |
| **Flow** | `render((maxcol,), focus)` | Calculates height automatically based on available width. | `Text`, `Edit`, `Button`, `CheckBox`, `Divider` |
| **Box** | `render((maxcol, maxrow), focus)` | Fills a explicit rectangular bounding box (both width and height). | `ListBox`, `SolidFill` |
| **Fixed** | `render((), focus)` | Controls its own exact width and height internally. | `BigText` |

### Absolute Layout Rules:

1. **The MainLoop Root Constraint:** `urwid.MainLoop(root_widget)` expects `root_widget` to be a **Box** widget.
2. **Filler Rule:** To display a **Flow** widget (like `Text` or `Edit`) as the root or inside a Box container, you **must** wrap it in `urwid.Filler(flow_widget, valign='top'|'middle'|'bottom')` to elevate it to a Box widget.
3. **Padding Rule:** To wrap a Flow widget to restrict its horizontal layout, use `urwid.Padding(flow_widget, align='left'|'center'|'right', width=...)`.
4. **ListBox Items:** `urwid.ListBox` is a **Box** widget, but it expects its child walker elements to be **Flow** widgets. Do not place Box widgets inside a standard ListBox walker.

---

## 2. Container Widgets & Sizing Specs

### A. Pile (Vertical Stacking)

Groups widgets vertically. Can mix Flow and Box widgets, but Box widgets *must* specify explicit constraints inside the layout list:

```python
# Syntax: urwid.Pile(widget_list)
pile = urwid.Pile([
    # Standard Flow widget (gets natural height)
    urwid.Text("Title:"),
    
    # Box widget wrapped with a relative weight
    ('weight', 1, urwid.ListBox(walker)),
    
    # Box or Flow widget wrapped with a fixed height
    ('fixed', 3, urwid.Filler(urwid.Text("Footer text")))
])

```

### B. Columns (Horizontal Splitting)

Groups widgets horizontally.

```python
# Syntax: urwid.Columns(widget_list, dividechars=0, focus_column=None)
cols = urwid.Columns([
    # Relative width ratio (weight)
    ('weight', 2, left_widget),
    
    # Fixed absolute character width
    ('given', 15, right_widget),
    
    # Flow widget gets natural layout
    flow_widget
], dividechars=2)

```

### C. Frame (Structured Desktop Layout)

A standard Frame layout containing a top header (Flow), middle body (Box), and bottom footer (Flow).

```python
# Syntax: urwid.Frame(body, header=None, footer=None, focus_part='body')
# Note: body MUST be a Box widget. header/footer must be Flow widgets.
frame = urwid.Frame(
    body=urwid.ListBox(walker),
    header=urwid.AttrMap(urwid.Text(" Application Title"), 'header_attr'),
    footer=urwid.Text(" [Q] Quit | [Tab] Navigate")
)

```

### D. Overlay (Floating Windows / Modals)

Overlays a foreground widget on top of a background widget.

```python
# Syntax: urwid.Overlay(top_w, bottom_w, align, width, valign, height)
# top_w: Box widget. bottom_w: Box widget.
overlay = urwid.Overlay(
    top_w=urwid.LineBox(dialog_inner), # Frame or Pile inside a LineBox
    bottom_w=main_layout,
    align='center', width=('relative', 60), # 60% of screen width
    valign='middle', height=('relative', 50) # 50% of screen height
)

```

---

## 3. Styling: The Color Palette & AttrMap

Urwid maps styles indirectly. Register a list of mappings (the "palette") with the `MainLoop`, then use `urwid.AttrMap` to attach style names to widgets.

### Palette Tuple Structure:

`('style_name', 'foreground_16', 'background_16', 'mono_attr', 'foreground_high', 'background_high')`

```python
PALETTE = [
    # Standard 16-color terminals
    ('normal', 'light gray', 'black'),
    ('header', 'white', 'dark red', 'bold'),
    
    # State-based styling: ('name', fg, bg, mono, fg_high, bg_high)
    ('focus_button', 'yellow', 'dark blue', 'bold'),
    ('edit_active', 'black', 'light gray', 'underline'),
]

# Apply styling to a widget: AttrMap(widget, normal_attr, focus_attr=None)
wrapped_text = urwid.AttrMap(urwid.Text("Styled text"), 'normal')
wrapped_button = urwid.AttrMap(urwid.Button("Action"), 'normal', focus_map='focus_button')

```

---

## 4. Keypress & Input Management

### Global Input Handler (Passed to MainLoop)

```python
def global_input_handler(key):
    # Escape sequences or normal character strings
    if key in ('q', 'Q', 'esc'):
        raise urwid.ExitMainLoop()

```

### Subclassing Widgets for Custom Keystrokes

If writing custom key behaviors inside a widget, you must override `keypress`. **Crucial**: If your widget does not handle the key, you **must** return the `key` argument unaltered so it bubbles up the hierarchy.

```python
class NavigationList(urwid.ListBox):
    def keypress(self, size, key):
        # Intercept keys
        if key == 'tab':
            # Handle keypress internally...
            return None # Return None to signal the key was consumed
            
        # Fall back to parent keypress behavior
        return super().keypress(size, key)

```

---

## 5. Event Signals API

Connect loosely coupled widgets (like buttons or state updates) using Urwid's signal dispatcher.

```python
# 1. Register signals on a custom widget class (must run at import/class level)
class CustomForm(urwid.WidgetPlaceholder):
    signals = ['submit', 'cancel']
    
    def on_submit_pressed(self, button):
        # Emit signal to notify listeners
        urwid.emit_signal(self, 'submit', self, self.get_data())

# 2. Connect instances to handler functions
def handle_form_submit(sender, data):
    # Action code here...
    pass

form_instance = CustomForm(...)
urwid.connect_signal(form_instance, 'submit', handle_form_submit)

```

---

## 6. Complete Boilerplate Design Pattern

Use this robust design pattern for building solid, expandable Urwid terminal layouts containing text inputs, button handlers, dynamic list contents, and clean loop terminations:

```python
import urwid

# Define our visual look-and-feel
PALETTE = [
    ('background', 'light gray', 'black'),
    ('header', 'white', 'dark blue', 'bold'),
    ('footer', 'black', 'light gray'),
    ('btn_normal', 'light gray', 'black'),
    ('btn_focus', 'black', 'light cyan', 'bold'),
    ('edit_normal', 'light gray', 'dark gray'),
    ('edit_focus', 'black', 'yellow', 'bold'),
]

class AppController:
    def __init__(self):
        # Dynamic container for state
        self.items_list = ["Item A", "Item B", "Item C"]
        self.walker = urwid.SimpleFocusListWalker([])
        self.populate_list()

        # Build listbox (Box Widget)
        self.listbox = urwid.ListBox(self.walker)
        
        # Build form panel (Flow Widgets -> wrapped in a Box Widget)
        self.input_field = urwid.AttrMap(
            urwid.Edit(caption="Add Item: ", edit_text=""),
            'edit_normal', 'edit_focus'
        )
        submit_btn = urwid.AttrMap(
            urwid.Button("Submit"), 
            'btn_normal', 'btn_focus'
        )
        urwid.connect_signal(submit_btn.original_widget, 'click', self.on_add_item)
        
        # Stack inputs into a Pile
        form_pile = urwid.Pile([
            self.input_field,
            urwid.Divider(),
            submit_btn
        ])
        # Force the Flow Pile into a Box using Filler
        form_box = urwid.Filler(form_pile, valign='top')

        # Arrange Layout: Listbox on left (weight 2), form panel on right (weight 1)
        layout_columns = urwid.Columns([
            ('weight', 2, urwid.LineBox(self.listbox, title="Dynamic Database")),
            ('weight', 1, urwid.LineBox(form_box, title="Editor Control"))
        ])

        # Master Frame Layout
        self.view = urwid.Frame(
            body=layout_columns,
            header=urwid.AttrMap(urwid.Text(" URWID SYSTEM CONTROLLER", align='center'), 'header'),
            footer=urwid.AttrMap(urwid.Text(" [Tab] Cycle Focus | [Esc / Q] Quit application"), 'footer')
        )

    def populate_list(self):
        """Re-render list entries from internal state."""
        self.walker.clear()
        for item in self.items_list:
            text_widget = urwid.Text(f"• {item}")
            # Wrap in AttrMap to support selection highlighting in listbox
            styled_widget = urwid.AttrMap(text_widget, None, focus_map='btn_focus')
            self.walker.append(styled_widget)

    def on_add_item(self, button):
        """Button click callback."""
        # Grab text directly out of the AttrMap wrapped Edit widget
        new_text = self.input_field.original_widget.get_edit_text().strip()
        if new_text:
            self.items_list.append(new_text)
            self.populate_list()
            # Clear input
            self.input_field.original_widget.set_edit_text("")

    def handle_keys(self, key):
        """Global UI Hotkey bindings."""
        if key in ('q', 'Q', 'esc'):
            raise urwid.ExitMainLoop()
            
        # Handle custom dynamic navigation keys
        if key == 'tab':
            # Loop widget focus inside the current columns
            pass

def main():
    app = AppController()
    loop = urwid.MainLoop(
        app.view, 
        palette=PALETTE, 
        unhandled_input=app.handle_keys
    )
    loop.run()

if __name__ == "__main__":
    main()

```