# one_life

One time value consumption helper.

_The author of this crate is not good at English._  
_Forgive me if the document is hard to read._

## What is this?

Core item of this crate is type `One`. It is a very simple wrapper.
It delegates all operations to the wrapped value as smart pointer.
However, if you use `One::take` and extract thw wrapped value,
most of the subsequent operations will panic.

## For readability

Internally, `One` is just newtype of [`Option`]. And its functionality
is more restricted than `Option`. However, because of this, we get the
following advantages in code reading and writing.

- In code reading, the reasons for wrapping becomes more clear.
- In code writing, unwrapping is no longer necessary.

## Other approaches

In some special cases, the following might be more useful than
`One::take` or [`Option::take`].

- [`mem::take`] - Only usable if field type implements [`Default`]
- [`mem::replace`] with [`mem::zeroed`] - Required `unsafe`, but highspeed

## Examples

Simple example.

```rust
use one_life::prelude::*;

let mut val = One::new("foo");
assert_eq!(val.to_uppercase(), "FOO");
assert_eq!(One::take(&mut val), "foo");
assert!(!One::exists(&val));
```

Common [`Drop`] example (with double meaning 😓).

```rust
use one_life::prelude::*;

let mut message_box = None;
let mut worker = Worker::new(&mut message_box);
assert_eq!(worker.message(), "I am a new worker!");
worker.do_hard_work();
assert_eq!(worker.message(), "I am buzy!");
worker.do_bullshit_work();
assert_eq!(message_box.unwrap(), "I am retired!");

struct Worker<'a> {
    message: One<String>,
    message_box: &'a mut Option<String>,
}

impl<'a> Worker<'a> {
    pub fn new(message_box: &'a mut Option<String>) -> Self {
        let message = One::new("I am a new worker!".to_string());
        Self {
            message,
            message_box,
        }
    }

    pub fn message(&self) -> &str {
        &self.message
    }

    pub fn do_hard_work(&mut self) {
        *self.message = "I am buzy!".to_string();
    }

    pub fn do_bullshit_work(mut self) {
        *self.message = "I am retired!".to_string()
    }
}

impl Drop for Worker<'_> {
    fn drop(&mut self) {
        *self.message_box = Some(One::take(&mut self.message));
    }
}
```

## History

See [CHANGELOG](CHANGELOG.md).

<!-- Links -->

[`Default`]: https://doc.rust-lang.org/std/default/trait.Default.html
[`Drop`]: https://doc.rust-lang.org/std/ops/trait.Drop.html
[`Drop::drop`]: https://doc.rust-lang.org/std/ops/trait.Drop.html#tymethod.drop
[`Option`]: https://doc.rust-lang.org/std/option/enum.Option.html
[`Option::take`]: https://doc.rust-lang.org/std/option/enum.Option.html#method.take
[`mem::replace`]: https://doc.rust-lang.org/std/mem/fn.replace.html
[`mem::take`]: https://doc.rust-lang.org/std/mem/fn.take.html
[`mem::zeroed`]: https://doc.rust-lang.org/std/mem/fn.zeroed.html
