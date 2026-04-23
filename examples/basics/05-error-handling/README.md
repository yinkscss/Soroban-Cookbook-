# Error Handling

This example demonstrates proper error propagation patterns in Soroban smart contracts using custom error types, `Result<T, E>`, the `?` operator, and explicit error conversion.

## Project Structure

```text
examples/basics/05-error-handling/
├── Cargo.toml
├── README.md
└── src/
    ├── lib.rs
    └── test.rs
```

## What This Example Shows

- Defining contract-level and domain-level error enums
- Returning `Result<T, Error>` for recoverable failures
- Propagating errors with the `?` operator across helper functions
- Converting lower-level errors into contract errors with `From`
- Verifying bubbling behavior and conversion in tests

## Key Concepts

### Contract Error Type

The contract defines a custom error enum using the `#[contracterror]` attribute:

```rust
#[contracterror]
#[derive(Copy, Clone, Debug, Eq, PartialEq)]
#[repr(u32)]
pub enum Error {
    InvalidAmount = 1,
    InsufficientBalance = 2,
    Unauthorized = 3,
}
```

### Domain Error + Conversion

```rust
#[derive(Copy, Clone, Debug, Eq, PartialEq)]
#[repr(u32)]
pub enum MathError {
    DivisionByZero = 10,
}

impl From<MathError> for Error {
    fn from(value: MathError) -> Self {
        match value {
            MathError::DivisionByZero => Error::InvalidAmount,
        }
    }
}
```

### Error Propagation with `?`

Contract functions return `Result<T, Error>` to handle failures gracefully:

```rust
pub fn transfer(amount: u64, balance: u64) -> Result<u64, Error> {
    Self::validate_transfer(amount, balance)?;
    Self::subtract_balance(amount, balance)
}
```

### Error Conversion and Bubbling

```rust
pub fn divide_checked(a: i128, b: i128) -> Result<i128, MathError> {
    if b == 0 {
        return Err(MathError::DivisionByZero);
    }
    Ok(a / b)
}

pub fn divide_with_conversion(a: i128, b: i128) -> Result<i128, Error> {
    Ok(Self::divide_checked(a, b).map_err(Error::from)?)
}
```

### Testing Errors

The test suite demonstrates success, conversion, and bubbling scenarios:

- `test_divide_checked_returns_domain_error()`
- `test_divide_with_conversion_maps_error_to_contract_error()`
- `test_error_bubbling_with_question_operator()`

## Build

From repository root:

```bash
cargo build -p soroban-error-handling-example
```

Or from this directory:

```bash
cargo build
```

## Test

```bash
cargo test -p soroban-error-handling-example
```

## Learn More

- [Soroban Error Handling Documentation](https://developers.stellar.org/docs/build/smart-contracts/errors-and-debugging/debug-errors)
- [Custom Contract Errors](https://developers.stellar.org/docs/build/smart-contracts/example-contracts/errors)
