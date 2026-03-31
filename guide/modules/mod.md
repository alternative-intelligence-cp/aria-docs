# mod — Defining Modules

## Module Declaration

```aria
mod math {
    pub func:add = int32(int32:a, int32:b) {
        pass (a + b);
    };

    func:helper = int32(int32:x) {   // private to module
        pass (x * 2);
    };
}
```

External module declaration (separate file):

```aria
mod crypto;
```

## Visibility

- `pub` — public, visible to importers
- Default — private, visible only within the module/file

## Related

- [use_import.md](use_import.md) — importing modules
