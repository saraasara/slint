<!-- Copyright © SixtyFPS GmbH <info@slint.dev> ; SPDX-License-Identifier: GPL-3.0-only OR LicenseRef-Slint-Royalty-free-1.1 OR LicenseRef-Slint-commercial -->

# Slint-python (Alpha)

[Slint](https://slint.dev/) is a UI toolkit that supports different programming languages.
Slint-python is the integration with Python.

**Warning: Alpha**
Slint-python is still in the very early stages of development: APIs will change and important features are still being developed,
the project is overall incomplete.

You can track the overall progress for the Python integration by looking at python-labelled issues at https://github.com/slint-ui/slint/labels/a%3Alanguage-python .

## Slint Language Manual

The [Slint Language Documentation](../slint) covers the Slint UI description language
in detail.

## Prerequisites

* Install Rust by following the [Rust Getting Started Guide](https://www.rust-lang.org/learn/get-started). If you already
  have Rust installed, make sure that it's at least version 1.70 or newer. You can check which version you have installed
  by running `rustc --version`. Once this is done, you should have the `rustc` compiler and the `cargo` build system installed in your path. This requirement will be removed before the final release of Slint for Python.
 * [Python 3](https://python.org/)
 * [pip](https://pypi.org/project/pip/)
 * [Pipenv](https://pipenv.pypa.io/en/latest/installation.html#installing-pipenv)

## Try it out

If you want to just play with this, you can try running our Python port of the [printer demo](../../examples/printerdemo/python/README.md):

```bash
cd examples/printerdemo/python
pipenv update
pipenv run python main.py
```

## Quick Start

1. Add Slint from the Git development branch to your Python project: `pipenv install "git+https://github.com/slint-ui/slint#subdirectory=api/python&egg=slint"`
2. Create a file called `appwindow.slint`:

```
import { Button, VerticalBox } from "std-widgets.slint";

export component AppWindow inherits Window {
    in-out property<int> counter: 42;
    callback request-increase-value();
    VerticalBox {
        Text {
            text: "Counter: \{root.counter}";
        }
        Button {
            text: "Increase value";
            clicked => {
                root.request-increase-value();
            }
        }
    }
}
```

1. Create a file called `main.py`:

```python
import slint
import appwindow_slint

class App(appwindow_slint.AppWindow):
    @slint.callback
    def request_increase_value(self):
        self.counter = self.counter + 1

app = App()
app.run()
```

4. Run it with `pipenv run python main.py`

## API Overview

### Instantiating a Component

The following example shows how to instantiate a Slint component in Python:

**`ui.slint`**

```
export component MainWindow inherits Window {
    callback clicked <=> i-touch-area.clicked;

    in property <int> counter;

    width: 400px;
    height: 200px;

    i-touch-area := TouchArea {}
}
```

The exported component is exposed as a Python class. To access this class, you have two
options:

1. Call `slint.load_file("ui.slint")`. The returned object is a [namespace](https://docs.python.org/3/library/types.html#types.SimpleNamespace),
   that provides the `MainWindow` class:
   ```python
   import slint
   components = slint.load_file("ui.slint")
   main_window = components.MainWindow()
   ```

2. Import the `.slint` file as module by treating it like a Python module where the `.slint` extension is replaced with `_slint`:
   ```python
   import slint # needs to come first
   from ui_slint import MainWindow
   main_window = MainWindow()
   ```

### Accessing Properties

[Properties](../slint/src/language/syntax/properties) declared as `out` or `in-out` in `.slint` files are visible as  properties on the component instance.

```python
main_window.counter = 42
print(main_window.counter)
```

### Accessing Globals

[Global Singletons](https://slint.dev/docs/slint/src/language/syntax/globals#global-singletons) are accessible in
Python as properties in the component instance:

```
export global PrinterJobQueue {
    in-out property <int> job-count;
}
```

```python
print("job count:", instance.PrinterJobQueue.job_count)
```

### Setting and Invoking Callbacks

[Callbacks](src/language/syntax/callbacks) declared in `.slint` files are visible as callable properties on the component instance. Invoke them
as function to invoke the callback, and assign Python callables to set the callback handler.

Callbacks in Slint can be defined using the `callback` keyword and can be connected to a callback of an other component
using the `<=>` syntax.

**`my-component.slint`**

```slint
export component MyComponent inherits Window {
    callback clicked <=> i-touch-area.clicked;

    width: 400px;
    height: 200px;

    i-touch-area := TouchArea {}
}
```

The callbacks in Slint are exposed as properties and that can be called as a function.

**`main.py`**

```python
import slint
import MyComponent from my_component_slint

component = MyComponent()
# connect to a callback

def clicked():
    print("hello")

component.clicked = clicked
// invoke a callback
component.clicked();
```

Another way to set callbacks is to sub-class and use the `@slint.callback` decorator:

```python
import slint
import my_component_slint

class Component(my_component_slint.MyComponent):
    @slint.callback
    def clicked(self):
        print("hello")

component = Component()
```

The `@slint.callback()` decorator accepts a `name` named argument, when the name of the method
does not match the name of the callback in the `.slint` file. Similarly, a `global_name` argument
can be used to bind a method to a callback in a global singleton.

### Type Mappings

The types used for properties in the Slint Language each translate to specific types in Python. The follow table summarizes the entire mapping:

| `.slint` Type | Python Type | Notes |
| --- | --- | --- |
| `int` | `int` | |
| `float` | `float` | |
| `string` | `str` | |
| `color` | `slint.Color` |  |
| `brush` | `slint.Brush` |  |
| `image` | `slint.Image` |  |
| `length` | `float` | |
| `physical_length` | `float` | |
| `duration` | `float` | The number of milliseconds |
| `angle` | `float` | The angle in degrees |
| structure | `dict` | Structures are mapped to Python dictionaries where each structure field is an item. |
| array | `slint.Model` | |

### Arrays and Models

[Array properties](../slint/src/language/syntax/types#arrays-and-models) can be set from Python by passing
subclasses of `slint.Model`.

Use the `slint.ListModel` class to construct a model from an iterable.

```js
component.model = slint.ListModel([1, 2, 3]);
component.model.append(4)
del component.model[0]
```

When sub-classing `slint.Model`, provide the following methods:

```python
    def row_count(self):
        """Return the number of rows in your model"""

    def row_data(self, row):
        """Return data at specified row"""

    def set_row_data(self, row, data):
        """For read-write models, store data in the given row. When done call set.notify_row_changed:"
        ..."""
        self.notify_row_changed(row)
```

When adding/inserting rows, call `notify_row_added(row, count)` on the super class. Similarly, removal
requires notifying Slint by calling `notify_row_removed(row, count)`.

