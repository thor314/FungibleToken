# Trying to make Fungible Token work
The Fungible Token example is currently the best I have to go on for how to
setup and run a simulation test on NEAR protocol. It needed some work to get up
and running, and still isn't compiling like it ought. This is my attempt to fix that.

Changes so far:
 1. In `build.sh`, no `flags.sh` exists
`./build.sh: line 4: ../flags.sh: No such file or directory`
-> Remove flags from `build.sh`.

2. Replace near_sdk dependencies in toml with:
```
[dependencies]
near-sdk = "2.0.0"

[dev-dependencies]
near-sdk-sim = { git = "https://github.com/near/near-sdk-rs.git", tag="2.1.0" }
```

3. No `PanicOnDefault`
-> remove PanicOnDefault, replace with manual panic
```rust
impl Default for FungibleToken {
  fn default()-> Self{
    env::panic(b"nope");
  }
}
```

4. `cargo build` and `build.sh` now run. What about tests?
`cargo test`
```
error[E0432]: unresolved import `fungible_token::FungibleTokenContract`
  --> tests/general.rs:10:5
   |
10 | use fungible_token::FungibleTokenContract;
   |     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ no `FungibleTokenContract` in the root

warning: unused doc comment
  --> tests/general.rs:14:1
   |
14 | /// Load in contract bytes
   | ^^^^^^^^^^^^^^^^^^^^^^^^^^ rustdoc does not generate documentation for macro invocations
   |
   = note: `#[warn(unused_doc_comments)]` on by default
   = help: to document an item produced by a macro, the macro must produce the documentation as part of its expansion

warning: unused import: `std::str::FromStr`
 --> tests/general.rs:5:5
  |
5 | use std::str::FromStr;
  |     ^^^^^^^^^^^^^^^^^
  |
  = note: `#[warn(unused_imports)]` on by default

warning: unused doc comment
  --> tests/general.rs:67:5
   |
67 |       /// Uses default gas amount, `near_sdk_sim::DEFAULT_GAS`
   |       ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
68 | /     let res = call!(
69 | |         master_account,
70 | |         contract.transfer(alice.account_id.clone(), transfer_amount.into()),
71 | |         deposit = STORAGE_AMOUNT
72 | |     );
   | |______- rustdoc does not generate documentation for statements

error[E0609]: no field `contract` on type `&_`
  --> tests/general.rs:68:15
   |
68 |       let res = call!(
   |  _______________^
69 | |         master_account,
70 | |         contract.transfer(alice.account_id.clone(), transfer_amount.into()),
71 | |         deposit = STORAGE_AMOUNT
72 | |     );
   | |_____^
   |
   = note: this error originates in a macro (in Nightly builds, run with -Z macro-backtrace for more info)

error: aborting due to 2 previous errors; 3 warnings emitted
```
Hard nope. Did anyone run these tests before pushing?

-> Remove the doc comments and the `use::str::FromStr`.

Still have to solve `use fungible_token::FungibleTokenContract;` and  no field
`contract` on type `&_`, but they look to be related.


TO BE CONTINUED

HERE BEGINS ORIGINAL UNTOUCHED DOCUMENTATION

# Fungible token

Example implementation of a Fungible Token Standard (NEP#21).

NOTES:
 - The maximum balance value is limited by U128 (2**128 - 1).
 - JSON calls should pass U128 as a base-10 string. E.g. "100".
 - The contract optimizes the inner trie structure by hashing account IDs. It will prevent some
    abuse of deep tries. Shouldn't be an issue, once NEAR clients implement full hashing of keys.
  - This contract doesn't optimize the amount of storage, since any account can create unlimited
    amount of allowances to other accounts. It's unclear how to address this issue unless, this
    contract limits the total number of different allowances possible at the same time.
    And even if it limits the total number, it's still possible to transfer small amounts to
    multiple accounts.

## Building
To build run:
```bash
./build.sh
```

## Testing
To test run:
```bash
cargo test --package fungible-token -- --nocapture
```

## Changelog

### `0.3.0`

#### Breaking storage change

- Switching `UnorderedMap` to `LookupMap`. It makes it cheaper and faster due to decreased storage access.
