# Enum-to-types

Macro for generating pseudo-enums for type-level programming.
This is somewhat like https://github.com/fmease/tylift but implemented with `macro_rules!` syntax.

## Install

```toml
[dependencies]
enum-to-types = "0.1.0"
```

## Example

```rust
use enum_to_types::enum_to_types;
use std::marker::PhantomData;

enum_to_types!(AccessLevel; User, Admin);

struct DataStorage<T: AccessLevel::AccessLevel>(i32, PhantomData<T>);

impl<T: AccessLevel::AccessLevel> DataStorage<T> {
     fn new(i: i32) -> Self {
         Self(i, PhantomData)
     }
}

trait ReadStorage<T: AccessLevel::AccessLevel> {
    fn read(&self) -> i32;
}

impl ReadStorage<AccessLevel::Admin> for DataStorage<AccessLevel::User> {
     fn read(&self) -> i32 {
         self.0
     }
}

impl ReadStorage<AccessLevel::User> for DataStorage<AccessLevel::User> {
     fn read(&self) -> i32 {
         self.0
     }
}

impl ReadStorage<AccessLevel::Admin> for DataStorage<AccessLevel::Admin> {
     fn read(&self) -> i32 {
         self.0
     }
}

impl ReadStorage<AccessLevel::User> for DataStorage<AccessLevel::Admin> {
     fn read(&self) -> i32 {
         panic!("You have no rights to read this");
     }
}

fn main() {
     let storage = DataStorage::<AccessLevel::Admin>::new(1);
     assert_eq!(<DataStorage::<AccessLevel::Admin> as ReadStorage<AccessLevel::Admin>>::read(&storage), 1);
     let storage = DataStorage::<AccessLevel::User>::new(5);
     assert_eq!(<DataStorage::<AccessLevel::User> as ReadStorage<AccessLevel::User>>::read(&storage), 5);
     // reading storage with `AccessLevel::Admin` by user will cause panic
}
 ```

This may look very verbose, but it gives a lot of flexibility.
Also, other examples can look less verbose.

