# pickle-mixin

[![PyPI-License](https://img.shields.io/pypi/l/pickle-mixin.svg?style=flat-square)](https://pypi.python.org/pypi/pickle-mixin/)
[![PyPI-Version](https://img.shields.io/pypi/v/pickle-mixin.svg?style=flat-square)](https://pypi.python.org/pypi/pickle-mixin/)

Makes un-pickle-able objects pick-able.

## Install

You can install it via pip

```
pip install pickle-mixin
```

## Running the tests

After installation, you can test it
```
python -c "import pickle_mixin; pickle_mixin.test()"
```
as long as you have [pytest](http://docs.pytest.org/en/latest/).

## Usage

### Pickle by initialisation

Suppose that you have a class whose objects are un-pickle-able or that would
demand a large amount of disk space or memory to be pickle-able.
``PickleByInit`` class lets you pickle object attributes via object
initialization.
Consider the following classes:
```python
class Foo(PickleByInit):
    def __init__(self, obj):
        super(Foo, self).__init__()
        self.obj = obj

class Bar(object):
    def __init__(self, filename):
        self.filename = filename

    def __getstate__(self):
        raise PicklingError

    def init_dict(self):
        return dict(filename=self.filename)
```
Trying to pickle as follows
```python
f = Foo(Bar('file.txt'))
pickle.dumps(f)
```
would raise a ``PicklingError``.
The following on the other hand would work:
```python
f = Foo(Bar('file.txt'))
f.set_signature_only_attr('obj')
pickle.dumps(f)
```
The un-pickling process of ``f.obj`` attribute happens via object
initialisation, passing the returned dictionary from ``init_dict``
as keyword arguments to ``Bar.__init__``.

### Mixing classes with and without slots

Pickling does not save attributes defined via ``__slots__`` in the following
case:
```python
class Foo(object):
    __slots__ = ['a']

    def __init__(self):
        self.a = 4


class Bar(Foo):
    def __init__(self):
        pass
```

``SlotPickleMixin`` fixes it:
```python
class FooMixin(object):
    __slots__ = ['a']

    def __init__(self):
        self.a = 4


class BarMixin(FooMixin, SlotPickleMixin):
    def __init__(self):
        FooMixin.__init__(self)
        SlotPickleMixin.__init__(self)

f = BarMixin()
o = pickle.dumps(f)
f = pickle.loads(o)
assert hasattr(f, 'a')
```

## Authors

* **Danilo Horta** - [https://github.com/Horta](https://github.com/Horta)

## License

This project is licensed under the MIT License - see the
[LICENSE](LICENSE) file for details
