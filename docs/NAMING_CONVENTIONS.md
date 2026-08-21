# XIOM Naming Conventions

Frozen at v0.46.0. Change requires a compiler team RFC.
(5c-R: cheap now, brutal to retrofit -- rustc lesson from `library/core/src/option.rs`)

## 1. Function Naming Grammar

| Prefix | Meaning | Example |
|--------|---------|---------|
| `new()` | Allocates and returns a fresh instance | `Vec[T].new()`, `Layout.new(size)` |
| `with_*()` | Constructor with pre-configured state | `Vec[T].with_capacity(10)` |
| `from_*()` | Conversion from another type | `Str.from_int(42)` |
| `as_*()` | Zero-cost view/reinterpretation | `buf.as_ptr()` |
| `to_*()` | Allocating conversion | `int.to_string()` |
| `into_*()` | Consuming conversion (moves self) | `result.into_ok()` |
| `try_*()` | Fallible operation returning `Result` or `Option` | `parser.try_parse()` |
| `is_*()` | Boolean predicate | `opt.is_some()`, `agent.is_idle()` |
| `has_*()` | Collection membership check | `map.has_key("foo")` |

## 2. Method Suffixes

| Suffix | Meaning | Example |
|--------|---------|---------|
| `_or()` | Provide a fallback value | `opt.unwrap_or(0)` |
| `_or_else()` | Call a fallback closure | `opt.unwrap_or_else(fn)` |
| `_or_default()` | Return type's default value | `opt.unwrap_or_default()` |
| `_unchecked()` | Skip safety checks (unsafe, opt-in) | `vec.get_unchecked(0)` |

## 3. Ownership & Mutability

| Pattern | Convention |
|---------|----------|
| `&self` | Read-only borrow (default for query methods) |
| `&mut self` | Mutable borrow (for mutation methods: `push`, `set`, `clear`) |
| `self` | Consuming move (for `into_*` methods) |
| `Type.method(self, ...)` | Static dispatch with explicit first param |

## 4. Types

| Category | Naming | Example |
|----------|--------|---------|
| Structs | PascalCase | `HttpHeaders`, `SocketAddr` |
| Enums | PascalCase | `Option`, `Result`, `JsonValue` |
| Traits/interfaces | PascalCase | `Comparable`, `Hash` |
| Type aliases | PascalCase | `type Meter = Int;` |
| Type parameters | Single uppercase letter | `T`, `K`, `V` |
| Constants | UPPER_SNAKE_CASE | `MAX_SIZE`, `DEFAULT_PORT` |

## 5. Modules & Files

| Convention | Example |
|----------|---------|
| Module name | `module xiom.collections` |
| File path | `stdlib/xiom/collections.xi` |
| One type/module per file | Convention, not enforced |
| `pub` exports are explicit | Everything private by default |

## 6. `#[must_use]` Annotations

Functions whose return value should not be silently discarded:

- All `Result[T, E]` returns
- All `Option[T]` returns
- All pure constructors (`new`, `from_*`)
- All `clone()` methods

## 7. Stability

| Annotation | Meaning |
|----------|---------|
| `stability: stable` | Backward-compatible forever |
| `stability: experimental` | May change or be removed |
| `stability: deprecated(since, reason)` | Will be removed; use alternative |
| `stability: internal` | Not part of the public API |

## 8. Error Codes

Reserved blocks:
- `X0000-X0099`: Parser diagnostics
- `X0100-X0199`: Checker diagnostics
- `X0200-X0299`: Borrow checker
- `X7000-X7999`: Contract diagnostics

See [docs/error_codes/README.md](./error_codes/README.md) for the full registry.
