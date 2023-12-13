# Observing Ordered Dictionary

The observing ordered dictionary is a datastructure for accessing child-elements by name or position. It is meant to be inherited (or composed: I inherit it in my applications, but it's almost composed given I override and rename every public method).

Parent objects will register themselves with their child elements, who can notify the parent object of changes to its (the child's) properties.

It is well, if not chaotically, tested.

NB: While `ood` supports iteration and `len()`, indexing by `[]` is not supported yet.

## Basic Use:

```python
o = ObservingOrderedDictionary(*items_to_add)
o.add_items(*items_to_add)
o.pop_items(*items_to_pop) # returns list
o.get_items(*selectors) # returns list
o.get_item(selector) # see Selectors section below
o.has_item(selector)
o.reorder_all_items([selectors])
o.move_items(*selectors, ...) # must have one of #before=, after=, position=, distance=
```

Children must inherit class `ChildObserved`, whose `__init()__` takes a `name=`, and will supply `.get_name` and `.set_name`:
```python
class Item(ChildObserved):
    pass

o = Item("test")
o.get_name() == "test" # True
o.set_name("test2")
o.get_name() == "test2"
```
As well as other internal \_functions used in parent-child communication.

### Selectors

A primitive selector is either an integer (the position of the child) or a string (the name of the child).

There is also a `class Selector()`, which is inherited by three other classes you can use from `ood.selectors`:

```python
import ood
import ood.selectors as s

# Class 1:
# Select which child precisely if children w/ same name are allowed.
# Ordered by insertion.
s.Name_I(child_name str, child_index int)

# Clas 2:
# Select children which they themselves have specified children.
s.Has_Child(selector) # 
```

Class 3:

All `ChildObserved` are also selectors which specify to return themselves. Example:
```python
import ood

child = ood.ObservedChild("A")
not_child = ood.ChildObserved("B")
parent = ood.ObservingOrderedDictionary(child)

parent.has_item(child)  # True
parent.has_item("A")    # True
parent.has_item(0)      # True, useless.

parent.has_item(not_child)  # False
parent.has_item("B")        # False
parent.has_item(1)          # False
```

### Configuration Settings:

`ood.exceptions` has `ErrorLevel.IGNORE`, `ErrorLevel.WARN` and `ErrorLevel.ERROR`. `True` is also `ERROR` and `False` is `IGNORE`.

The default values are shown below, they can be overridden:

```python
import ood.exception as err

# Do non-existing indices to get_item(s), pop_items produce errors or empty array (or None)?
err.StrictIndexException.default_level = err.ErrorLevel.IGNORE

# Can one child have multiple parents?
err.MultiParentException = err.ErrorLevel.ERROR # WARN not possible

# Can multiple children of one parent have the samse name?
err.NameConflictException = err.ErrorLevel.ERROR # WARN not possible

# Do we warn or error if you add the same child twice?
err.RedundantAddException = err.ErrorLevel.WARN
```

## Extending

Your object can inherit both `ObservingOrderedDictionary` and `ChildObserved` to be both a parent and child object. If you override class variables `_type` and `_child_type`, exceptions generated will use those terms instead of "item" when raising exceptions.

```python
class RedNodes(ood.ObservingOrderedDictionary, ood.ChildObserved):
    _type="RedNode"
    _child_type="Node"
    def __init__(self, *args, **kwargs):
        _my_vars = kwargs.pop("key", "default_value") # Popping is important! Don't pass weird stuff to ood!
        super().__init__(*args, **kwargs)
        ... # Do whatever you want

    def get_node(self, selector, **kwargs):
        return super().__get_item(self, selector, **kwargs)
    ... # etc
```

### Adding more observed variables:

Children can call `_notify_parents(parameter=..., old_parameter=...)`, where parameter would be the parameter that changes.
For example, `set_name("new_name")` calls `_notify_parents(self, name="new_name", old_name="old_name").

On the parent side, you must implement 

```python
o._child_options(self, child, **kwargs) 
o._child_update(self, child, **kwargs)
```

`_child_options()` should `raise` an exception, and the child should understand that the parameter change will not be accepted. (this is used in case the configuration is set that two children cannot have the same name).

```python
def set_name(self, name):
    old_name = self.get_name()
    self._name = name
    try:
        _notify_parents(self, old_name=old_name, name=name)
    except:
        self._name = old_name
        raise Exception("Can't change name!")
```