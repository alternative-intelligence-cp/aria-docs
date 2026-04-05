# Rules and Limits

## Defining Rules

Rules declare value constraints that the SMT solver can verify at compile time.

```aria
Rules<int32>:Positive     = { $ > 0 };
Rules<int32>:Bounded      = { $ >= 0, $ <= 1000 };
Rules<int32>:NonZero      = { $ >= 1, $ <= 1000 };
Rules<flt64>:UnitInterval = { $ >= 0.0, $ <= 1.0 };
```

`$` refers to the value being constrained. Multiple conditions are ANDed together.

## Applying Limits

Use `limit<RulesName>` on variable declarations to apply constraints:

```aria
Rules<int32>:Positive = { $ > 0 };
Rules<int32>:Bounded  = { $ >= 0, $ <= 1000 };

func:example = int32() {
    limit<Positive> int32:x = 42;      // Proven: 42 > 0
    limit<Bounded>  int32:y = 500;     // Proven: 500 in [0, 1000]
    limit<Positive> int32:z = -1;      // Disproven: -1 > 0 is false → compile error
    pass x + y;
};
```

With `--verify --prove-report`:
```
[limit] Proven:  'x' satisfies Positive
[limit] Proven:  'y' satisfies Bounded
[limit] Disproven: 'z' violates Positive — counterexample: $ = -1
```

## Rules Consistency

The solver checks that Rules are satisfiable — if no value can ever match, it's an error:

```aria
Rules<int32>:Impossible = { $ > 10, $ < 5 };   // Error: unsatisfiable
```

## Rules Narrowing

When one Rules is a subset of another, the compiler proves subsumption automatically:

```aria
Rules<int32>:Positive       = { $ > 0 };
Rules<int32>:SmallPositive  = { limit<Positive>, $ < 100 };

// SmallPositive inherits $ > 0 from Positive, adds $ < 100
// Passing SmallPositive where Positive is expected: always safe
```

## Rules Propagation

When all callers of a function pass `limit<>`-constrained arguments, constraints
propagate to the callee for interprocedural optimization:

```aria
Rules<int32>:Safe = { $ >= 0, $ <= 100 };

func:process = int32(int32:val) {
    pass val * 2;     // Overflow check eliminated: max 100*2 = 200 < INT32_MAX
};

func:main = int32() {
    limit<Safe> int32:a = 50;
    int32:result = raw process(a);
    exit 0;
};
```

## Range Inference (v0.14.1+)

The solver infers value ranges through assignments and control flow, even without
explicit Rules annotations:

```aria
Rules<int32>:Small = { $ >= 1, $ <= 100 };

func:example = int32() {
    limit<Small> int32:a = 10;
    limit<Small> int32:b = 20;
    int32:sum = a + b;         // Range inferred: [2, 200] → overflow-free
    int32:doubled = sum + sum; // Range: [4, 400] → still overflow-free
    pass doubled;
};
```

The range analyzer propagates ranges through:
- Arithmetic (`+`, `-`, `*`, `/`, `%`)
- Assignments (intermediate variables inherit ranges)
- Control flow (if-else narrows ranges on each branch)

## Writing Solver-Friendly Code

1. **Apply Rules at boundaries** — where data enters the system
2. **Use narrow constraints** — `{ $ >= 0, $ <= 100 }` is more provable than `{ $ > 0 }`
3. **Constrain divisors** — `{ $ >= 1, $ <= N }` enables division-by-zero elimination
4. **Name your Rules** — meaningful names document intent and aid debugging
5. **Compose with `limit<Parent>`** — build hierarchies of increasingly narrow constraints

## Next

- [Contracts](03_contracts.md) — function-level verification
- [Optimizations](04_optimizations.md) — turning proofs into performance
