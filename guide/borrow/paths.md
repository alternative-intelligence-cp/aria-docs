# Field Paths

The borrow checker is *path sensitive*: two borrows on different fields of
the same struct are independent loans, not conflicting ones.

## Sibling fields are disjoint

```aria
struct:Pair = {
    int32:a;
    int32:b;
};

func:main = int32() {
    Pair:pair = Pair{a: 10, b: 20};
    $$m int32:ra = pair.a;     // loan on path pair.a
    $$m int32:rb = pair.b;     // loan on path pair.b — disjoint, OK
    exit pair.a + pair.b;       // 30
};
```

Two `$$m` loans coexist because the paths `pair.a` and `pair.b` do not
overlap. (K test `077`.)

## Same path conflicts

Two loans on the same exact path conflict, just like for plain variables:

```aria
$$m int32:r1 = pair.a;
$$m int32:r2 = pair.a;       // ARIA-023: conflicting borrow on `pair.a`
```

(K test `078`.)

## Nested fields

The same rules extend to nested fields, up to three levels deep — the
syntax depth supported by both the parser and the K model:

```aria
struct:Leaf = { int32:x; int32:y; int32:z; };
struct:Box  = { Leaf:leaf; int32:tag; };

func:main = int32() {
    Box:box = Box{leaf: Leaf{x: 10, y: 20, z: 30}, tag: 1};
    $$m int32:rx = box.leaf.x;
    $$m int32:ry = box.leaf.y;
    $$m int32:rz = box.leaf.z;     // three disjoint loans, OK
    exit box.leaf.x + box.leaf.y + box.leaf.z;   // 60
};
```

(K tests `091`, `143`.)

## Parent-child overlap

A loan on a parent path **does** overlap a loan on any of its descendants:

```aria
$$m int32:rx     = box.leaf.x;
$$m Leaf:rleaf   = box.leaf;   // ARIA-023: rleaf covers rx's path
```

(K test `093`.)

This is the only path rule that surprises people: `box.leaf` is *not*
disjoint from `box.leaf.x` even though the field names differ.

## Sibling assignment is fine

If only the borrow path is held, the host's *other* fields can still be
assigned to directly:

```aria
$$m int32:ra = pair.a;
pair.b = 99;       // OK — pair.b is not borrowed
```

(K test `079`.)

But assigning to the borrowed path itself is a conflict:

```aria
$$m int32:ra = pair.a;
pair.a = 99;       // ARIA-026: assignment through borrowed path
```

(K test `080`.)
