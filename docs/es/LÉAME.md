# Diccionario Ordenado que Observa 
(Observing Ordered Dictionary)

El dicionario es una estructura de datos para acceder a elementos contenidos por nombre o posición. Se cree para ser heredado. (o compuesto: yo lo herado en mis aplicaciones, pero es casi compuesto como yo sobrescribo/anulo y renombro cada método público)

Los objetos padres (`Observer()`) se registrarán con sus objetos secundarios, los hijos (`Observed()`), que avisan al objeto padre de cambios a sus (de lo hijo) propriedades. Los dos clases se combinan en `Item(Observer, Observed)`.

Está bien, si no caóticamente, probado.

Nota: si bien `Observer()` incluye iteración y `len()`, no se indexa por `[]`

## Uso Básico

```python
import ood

# Inicializar
o = ood.Observer(*para_agregar)

o.add_items(*para_agregar)
o.pop_items(*para_echar) # return list

# Acceder (nota sección "Selector" abajo)
o.get_items(*selectores) # return list
o.get_item(selector) # return item o None
o.has_item(selector)

# Reorganizar todo
o.reorder_all_items([selectors])

# Mover uno(s)
o.move_items(*selectors, ...) # requiere un de before=, after=, position=, distance=
# (antes, después, posición, distancia)
```

Secundarios debe heredar `class Observed()` cuyo `__init__(self, **kwargs)` quiere un argumento de `name`. Proveerá métodos `get_name()` y `set_name()`

```python
import ood

o = Observed(name = "SoyYo")
o.get_name() == "SoyYo" # True
o.set_name("FooBar")
o.get_name() == "FooBar" # True
```
También proveidos son otras  `_funcionas()` internas que se usa en communicación entre objetos y los hijos.

### Selectores

Un *selector* primitivo es o un `int` (la posición del hijo) o un `str` (el nombre).

También hay una `class Selector()`, que se heredan por 3 otras clases útiles (dos de `ood.selectors`)

#### Clases 1, 2

```python
import ood
import ood.selectors as s

# Clase 1:
# Si se permite hijos con los mismos nombre, eligir por índice también
# (Ordenado por by introducción)
s.Name_I(hijo_nombre str, hijo_índice int)

# Clase 2:
# Producir hijos que tengan sus mismos el selector
s.Has_Child(selector) # 
```

#### Clase 3:

Todo `Observed()` es un selector que devuelve su mismo. Ejemplo:
```python
import ood

hijo = ood.Observed("A")
no_hijo = ood.Observed("B")
padre = ood.Observer(hijo)

parent.has_item(hijo)   # True
parent.has_item("A")    # True
parent.has_item(0)      # True, inútil.

parent.has_item(no_hijo)    # False
parent.has_item("B")        # False
parent.has_item(1)          # False
```

### Configuración

`ood.exceptions` tiene `ErrorLevel.IGNORE`, `ErrorLevel.WARN` y `ErrorLevel.ERROR`. `True` también `ERROR` y `False` es `IGNORE`.

Los defectos se muestran abajo, se pueden sobreescribir.

```python
import ood.exception as err

# ¿Si el índice no se existe, se recibe error o lista vacía (o None)?
err.StrictIndexException.default_level = err.ErrorLevel.IGNORE

# ¿Se puede tener más de un padre, un hijo?
err.MultiParentException = err.ErrorLevel.ERROR # WARN no posible

# ¿Se puede tener el mismo nombre, unos hijos?
err.NameConflictException = err.ErrorLevel.ERROR # WARN no posible

# ¿Se avisa, se ignora, o se leventa error si se agrega el mismo secungadrio dos veces? 
err.RedundantAddException = err.ErrorLevel.WARN
```


## Extender

Si sobreescribir los variables del clase `_type` y `_child_type`, errores generados usarán esas palabras en vez de "item".

```python
class NodoRojo(ood.Item):
    _type="NodoRojo"
    _child_type="Nodo"
    def __init__(self, *args, **kwargs):
        _mi_vars = kwargs.pop("clave", "valor_defecto") # Pop es importante! No pasen cosas raras!
        super().__init__(*args, **kwargs)
        ... # Lo que quiera

    def get_nodo(self, selector, **kwargs):
        return super().get_item(self, selector, **kwargs)
    
    ... # etc
```

### Mejorar las variables observadas:

Secundarios llaman `_notify_parents(parámetror=..., old_parámetro=...)`, donde parámetro es lo que cambia.
Por ejemplo, `set_name("nombre_nuevo")` llama `_notify_parents(self, name="nombre_nuevo", old_name="nombre_antiguo")`.

Por el padre, se debe implementar:

```python
o._child_options(self, hijo, **kwargs) 
o._child_update(self, hijo, **kwargs)
```

`_child_options()` debe `raise` (leventar) un error, y el hijo debe entender que el cambio de parámetro no se permitirá. (Se usa en el caso que haya la configuración de qué cada hijo tenga un nombre único) 

```python
def set_name(self, name):
    old_name = self.get_name()
    self._name = name
    try:
        _notify_parents(self, old_name=old_name, name=name)
    except:
        self._name = old_name
        raise Exception("No puedo!")
```
