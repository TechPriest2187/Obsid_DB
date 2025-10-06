
#Learning material notes on Rust Programming Language

# Variables
Rust-specific variable concepts 
#### Shadowing
If we define variable two variables in the same scope with the same name, second variable will **shadow** first one, but they still would have distinct values and first variable would still be available:
```rust
fn main() {
	let x = 10;
	let x_ref = &x;
	let x = *&x + 10; // shadowing here

	println!("new_x: {x}");
	println!("old_x: {}", *x_ref);
}
```
#### Difference between `const` and `static`
The `const` does not really have a place in memory. It's like an alias for a magic value. With `const`, all computations can be in compile time (see [[Rust Compile-Time]])

#### [[Rust Ownership]]
In rust, every value has exactly one owner. If I define `let x = 10;`, then `x` will be the owner of value `10`. Every time value is moved (function call, push to vector) it's ownership transfers to the new location and therefore value must be accessed thru the new location.
##### Copy
Some types bypass this rule by implementing `Copy` trait. This means that every time value was supposed to move, it will be copied instead. Most primitive types are `Copy`, because it does not make sense to get reference to them as reference size is might be bigger than size of those types.
```rust
fn main() {
	let x1 = 42;
	let x2 = Box::new(10);
	{
		let z = (x1, x2); 
		// x1 and x2 drops here, because they are moved
	}
	let x3 = x1; // x1 was not dropper, because primitive types implement `Copy`
	// let x4 = x2; 
	// ^ but this won't work because Box is not `Copy` and was moved
}
```

### [[Rust References]]
Available two types: **shared** and **exclusive**. Differences are that there could be multiple **shared** references and only one **exclusive**. Also, **shared** are read-only and **exclusive** are mutable. Rust contract guarantees that values behind **shared** references will not be changed while reference is alive, and this enables some advanced optimizations ([[Alias Analysis]]). Therefore, there could not be **shared** and **exclusive** references at the same time. ^306c56

- Compiler assumes that there are no other threads accessing target value, whether thru shared or exclusive reference
	```rust
	// Rust contract guarantees that this assertion is keeped
	fn cache(input: &i32, sum: &mut i32) {
		*sum = *input + *input;
		assert_eq!(*sum, 2 * *input);
	}
```

- Rust references enable unique and interesting optimizations. For example:
```rust
fn noalias(input: &i32, output: &mut i32) {
	if *input == 1 {
		*output = 2;
	}
	if *input != 1 {
		*output = 3;
	}
}
```

Rust references contract states that value behind a shared reference won't be changed while reference is alive, and therefore `input` and `output` should point to the different memory locations. This means that `input` can not be aliased with `output` and that those two conditions could be merged into single if-else.

- References can be made pointing to another value:
```rust
let x = 10;
let y = 20;
let mut r = &x;

// to access value pointed by r we need to dereference it (*), but if it is not dereferenced, that means that we are operating on r itself
r = &y;
```

- With a `&mut T` reference we can do whatever we want with target value, except of moving: in order to move value out-of mutable reference, we need to leave something in-place instead. This is because otherwise value owner would free already moved value and this is [[Double-free]] error.

#### [[Rust Lifetimes]]
Lifetimes are needed to ensure that references would not out-live it's referent. 
Lifetimes are annotated as `'a` and in most cases compiler can **infer** them.

*Lifetimes are the name for the block of code for which the reference must be alive*

- Lifetimes can have holes
```rust
let mut x = Box::new(42);
let r = &x;

if rand() > 0.5 {
	*x = 10;
} else {
	println!("{}", r);
}
```
	Borrow checker always tries to find the shortest possible lifetime. I was surprise by this example, because I thought that `r` is still in scope of if block, and therefore x can't be mutated. But borrow checker traces use of reference back to the creation, and it sees no conficting flows, because shared reference is accessed in another possible flow and therefore accepts this code. If there was a statement that uses `r` after if-else block, than lifetime would be extended up to this line and therefore mutation of *x* would violate shared reference contract.

I should think of lifetimes as flow, because parallel flows could exist at the same time.

- Lifetimes does not have to be valid for entire blocks. They don't capture all the region, only uses of reference. In other words, they can have *holes*:
	```rust
	fn main() {
	    let mut x = Box::new(42); 
	    let mut z = &x;
	   
	   for i in 0..100 {
	       println!("{}", z); // lifetime spans here
	       x = Box::new(i); // moving out of z and therefore creating hole in lifetime
	       z = &x; // lifetime is kick-started there, so it is still valid for the next iteration
	   }
	}
	```

#### Rust lifetime variance
Lifetime are subject of [[Covariance and invariance]] rules. But, in contrast to the [[Object-oriented Principles]] languages, object of variance are **lifetimes**.

In brief, variance describes relationships between supertype and subtype in generic types. But Rust doesn't have inheritance, so only lifetimes obey to these rules. For example, `'static` is a subtype of arbitrary lifetime`'a`.  This is because for type to be subtype of supertype, it needs to be at least as "useful" as supertype. 
In context of lifetimes, it means that subtype MUST BE at least the size of supertype. And because `'static` is the longest lifetime you can get, in essence it is a subtype of every other lifetime.

There are 3 possible relationships variance's between types:
- **Covariance** (Variance passes-thru). Formally, this means that if T in `F<T>` is subtype of U, `F<T>` is subtype of `F<U>`.  AKA subtype can be placed in locations, where supertype is expected. 
	 - In Rust, for example, that means that we can pass `&'static u32` where `&'a u32` is expected, because '`static'` is subtype of `'a` and `&'a T` is **covariant** in `'a`.
		 - If T is covariant in U, that means that it allows changing U to it's subtype
- **Contravariance**. Reversed relationship of **covariance**. Formal definition: if T in `F<T>` is subtype of U, `F<U>` is subtype of `F<T>`. Comes up in Rust function's.
	 ```rust
	 // more strict, less useful
	 fn x(arg: &'static str)
	 // lest strict, more useful
	 fn x2(arg: &str)
	```
	- Essentially this means that I can pass functions with shorter lived arguments to function types, where longer lived function argument types is expected:
		```rust
		let x: impl Fn(&'static str) -> ();
		let y = |x: &str| {}; // here x is of arbitary lifetime 'a, not 'static
		x = y; // this works because of contravariance and essentially means that function contract is upholded, because 'static lives longer than function needs
		```

#### Covariance inheritance
Generic types inherent convariance of their type argument
# Cargo
Rust has an ultimate build system and package manager. This note won't cover all the basic features of Cargo, but the most interesting one's.

## Features
Projects can be divided by conditional compilation with features. Features are additive by nature, meaning that they should not change public interface, only append to it.
#### Features basic format
```
// Cargo.toml
name = "foo"

[features]
macros = ["syn"]

[dependencies.syn]
version = "1"
optional = true
```
Features of the crate are defined in [[TOML|toml's]] array specifier `[features]`.  The array on the right hand side of assignment denotes optional dependencies enabled (which should be specified with `optional = true` flag).
By default Cargo creates feature with the same name for every optional dependency, so features can't be named after dependencies.
#### Features with features in dependencies
Features can enabled features of their optional dependencies with `<dep>/<features>` syntax.

```
[features]
macros = ["syn/printtable"]
```
#### Opt-out features
By default features are opt-in, meaning user must explicitly specify in he wants that feature to be enabled. Sometimes feature are used so common, that it makes sense to provide opt-out option to exclude it from compilation.
```
[features]
macros = ["syn"]
default = ["macros"]
```
# Rust Types

### Alignment and layout
Rust aligns fields in structures on their natural alignment: this means that field is aligned on it's size.
Rust doesn't make much guarantees about type layout:
- Fields can be reordered
- Structs with identical fields can have their fields in different orders

To request guarantees about layout, Rust supports `#[repr]`
There are three possible repr-attribute values:
- `#[repr(C)]`: Generates C-layout, which is stable and not an object to changes:
	- Fields in memory are ordered the same as in source
	- Fields are naturally aligned
	- The struct itself is aligned on highest alignment across it's fields
	- Can insert padding between fields if one of them can't be naturally aligned
- `#[repr(transparent)]`: Can be placed only if struct has one field. Means that struct will have same layout as it's field. *Useful mostly for NewType pattern*
- `#[repr(packed)]`: Disables alignment at all

`#[repr(packed(x))]` or `#[repr(align(x))]` can be used to decrease or increase the representation of following field or type. If x is unspecified in `packed` than it assumed to be zero.

For enums there is also a primitive representation: for example `#[repr(u8)]`. This means that enum would be the same size as primitive and could hold only 2^size_of(primitive)\*8 dicriminators.

#### Complex types layout
- Tuple - same like a struct with fields of the same type in the same order
- Array - continuous sequence of elements without padding
- Union - layout is choosed independently across each variant. Alignment is the maximum from all variants
- Enumerations - same as union, but contains hidden field discriminator

#### Dynamically-sized types
In contrast to the other languages, Rust supports most of the compile-time. But in order support some features, something must be unknown.

Example of this is `Sized` trait. This trait means that size of type is known on compile time. In fact compiler requires types implementing this trait almost everywhere: in function arguments, return types, variables. 

But sometimes we does not know size of type. Common examples are trait objects (`dyn`) and slices (`[T]`). To bridge the gap, rust uses **wide pointers**. Essentially, they are the same pointers but with extra `usize` field for additional information. When you take a reference to DST, compiler automatically construct wide pointer.


> [!NOTE] Automatic bound
>  Bound on sized is included automatically in every `where` statement. It is possible to specify that `Sized` is not required, either with `?Sized` or `!Sized`. The former syntax means "may be or may be not" and the latter "Not Sized".

### Unions
...
### Generic types
Every generic types comes thru the `monomophirzation` process, meaning that type and it's impl blocks are copied for every generic instantiation. 

> In order to avoid some of monomorphization overhead, you can move shared behavior in separate method outside of generic impl block.  
## Traits
### Coherence property and orphan rule
Traits and types are subject for coherence property in rust: coherence property makes it impossible for type to have ambiguous implementation of trait. That is: the type can't have two implementations of the same trait.

The real problem come with multi-crate environment: how can we ensure that this property upholds for the shared type between 2 crates? Answer: Orphan rule.

> **To implement trait for type, either trait or type must be local to your crate**

#### Fundamental types
Some types should ignore orphan rule - they are called fundamental (&, &mut, Box).
They are marked with `#[fundamental]` attribute and effectively erased before orphan rule checking.

#### Exceptions
In order to support some more complex implementation (like `impl From<MyType> for Vec<MyType>)`, orphan rule has an exception:

`impl<T0, T1... Ti> for ForeignType<P0, Pi...>` is allowed as long as one of the `P` is local type, and no types before such are `Ti`

### Object safety
Not all objects that implement traits can be turned to trait objects. Trait itself must be object safe.
In order to be object safe, trait methods must not:
- Use `Self` type in any way
- Have generic methods
- Have static methods

If some methods requires `Self`, you can place a bound `Self: Sized` especially on that method. Such method are not included when checking for object safety.

# Unsafe Rust
Unsafe Rust is a language tool to enforce [[Invariants|invariants]] which can't be checked statically by compiler (e.g. borrow checker can't verify that value if borrower for it's lifetime).

## `unsafe` block and `unsafe fn`
In Rust, `unsafe` keyword has two meanings, which dependent on the context of usage:
- If used as block marker (e.g. `unsafe {...}`) states that safety guarantees was ensured to be uphold by the code author
- if uses as function marker (.e.g. `unsafe fn might_segfault() {...}`) states that caller must ensure that function's safety contract is held by caller
	- `unsafe fn` can be called only from `unsafe {...}` 

### `unsafe` block abilities
There are number of operations that can be executed only in `unsafe` block. They include low-level operations safety of which compiler can't prove, but which are still useful in number of contexts.

> [!NOTE] Unsafe is not a way to circumvent Rust rules 
> `unsafe` is a way to guarantee that code is *safe* to use, that programmer ensured carefully that there are no undefined behaviour

- Dereferencing [[Rust#Raw Pointers|Raw Pointers]]
- Accessing [[Rust#Rust Types#Unions|Unions]] fields
- Accessing mutable static's
- Accessing external static's

## Raw Pointers
Much like references, but do not have lifetimes and are not subject to [[Rust#^306c56|Rust's reference rules]].
There are two kinds of raw pointer: `*const T` and `*mut T`. The only difference noted in language specification is that `T` is invariant in `*mut T`, where in other kind it's covariant. But common sense one says that `*const T` is immutable, where `*mut T` is mutable.

Aside from raw pointers core types, there are number of raw pointer-like type in standard library. One notable of them is `std::ptr::NonNull<T>`, which states that pointer inside of it can never be null. It benefits from cool compiler optimization called [[Niche Optimization]], for example in `Option<std::ptr::NonNull<T>>` compiler can optimize `None` variant to be null, because pointer itself can never be null.

> Raw Pointers can't be dereferenced outside of `unsafe` block, but can be created in any place in program.

Raw Pointer are useful in situations when:
- You can't name a lifetime (e.g. self-referential data structure)
- Pointer arithmetic (.offset(), .add(), .sub())
	- **NOTE** it's an memory error to make pointer point beyond it's allocation
- Accessing highly-optimized in space data structures
- Working with FFI
- Constructing Rust core types (like slice's)

### `unsafe` correct usage rules
Because Rust compiler assumes that all code in unsafe is valid, it's perfectly in charge to add optimizations. Those optimizations in turn can cause [[Undefined behavior]] if Rust's guarantees and contract is not upheld. Below is a list (incomplete) of assumptions that Rust makes about code.

- Rust assumes that type's that hold memory via pointer (`Box`, `Rc`) has a unique memory: meaning that exact same memory chunk is not referenced or aliased elsewhere, unless this memory is explicitly accessed thru the shared reference.
- Some Rust primitive types have a restriction on which values they can hold. For example, despite `bool` being size of 1 byte, the only allowed values it can hold is `0x0` or `0x1`. If use unsafely make it store any other value, or cast a reference to bool which holds others than specified values, it is a breach of Rust's contract.
- All Rust values must be valid. It's a breach of contract otherwise, e.g. there is no possibility of uninitialized memory:
	- If needed, you can rely on `std::mem::MaybeUninit<T>` to have a uninitialized `T`, which you must manually initialize. In this case, compiler won't rely on validity of value until to the `MaybeUninit::assume_init` call.
- When using unsafe code, you should be aware of panics. Particularly, if some statement panics while in unsafe code, the [[Stack Unwiding]] process is launched. This means that all variables will be dropped in their order. You must be sure that drop is safe in any point of panic.
- It is unsafe to cast between types with `#[repr(Rust)]` layout, because compiler gives no guarantees about those type layouts. It's unsafe to cast even between zero-sized structs.

# FFI
Main [[FFI]] usage is thru `extern` keyword. This keyword defines symbol as external to this crate, and therefore it is linked with other object files. However, it can be perfectly used with Rust code as well, if Rust code is compiled as dynamic or static library.

## Name Mangling
[[Name Mangling]] is process of naming symbol pseudo-randomly in objects files. It's done to distinguish equivalently named, but different symbols. For example, there can be functions with same name in one crate, but in different modules.

Because externally-linked applications can't predict pseudo-random symbol names, it breaks compatibility with them.
Therefore, there is `#[no_mange]` [[attribute|Rust#Attribute]] in Rust, which turns off it.

> [!NOTE] Exporting Symbols from Rust
> All symbols are exported in objects file if they have `#[no_mangle]` attribute. It doesn't matter whenever symbol is public or not, it would be nevertheless exported to object file

# Extern Block
Basically marks all symbols and functions inside this block as externally linked.
```rust
extern {
	fn curl_easy_init() -> CURLCode
	fn curl_easy_perform(handle: *const CURLHandle) -> CURLCode

	static curl_version: cuint
}
```

## [[Calling convention]]
Obviously needed if working with system or some constrained API's. 

Syntaxis looks like
```rust
extern "C" fn hello_world(arg: i32);
```

Also supported by [[Extern Block's|Rust#Extern Block]]

# Rust Tricks

## Enum with variants/tuple struct as function
You can use enum fielded variant or tuple struct can be used as function from their members to their instances:
```rust
enum A {
	B(i32),
	C
}

fn main() {
	let data = vec![1, 2, 3, 4];
	let formatted_data = data.iter().map(A::B).collect();
}
```
