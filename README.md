# Web3 Substrate Blockchain Tutorial

## 🚀 Step 1: Prerequisites

Before you begin, ensure you have:

- Installed the Rust toolchain
- Cloned the Substrate Node Template:

  ```bash
  git clone https://github.com/substrate-developer-hub/substrate-node-template
  ```

- Built dependencies:

  ```bash
  cargo build --release
  ```

---

## 🧱 Step 2: Writing a Custom Pallet

We'll modify the default pallet to store a custom message on the blockchain.

### 📂 Location

`pallets/template/src/lib.rs`

```rust
// 1. Basic imports from FRAME (Framework for Runtime Aggregation of Modularized Entities)
use frame_support::{dispatch::DispatchResult, pallet_prelude::*};
use frame_system::pallet_prelude::*;

// 2. Declare our pallet module
#[frame_support::pallet]
pub mod pallet {
    use super::*;

    // 3. Config trait for configuring pallet dependencies
    #[pallet::config]
    pub trait Config: frame_system::Config {}

    // 4. Define pallet storage to hold a message
    #[pallet::storage]
    #[pallet::getter(fn stored_message)]
    pub type StoredMessage<T> = StorageValue<_, Vec<u8>, OptionQuery>;

    // 5. Declare callable functions (extrinsics)
    #[pallet::call]
    impl<T: Config> Pallet<T> {
        /// Set a message on the blockchain
        #[pallet::weight(10_000)]
        pub fn set_message(origin: OriginFor<T>, message: Vec<u8>) -> DispatchResult {
            let _sender = ensure_signed(origin)?;  // Ensure the sender is a valid signed user
            <StoredMessage<T>>::put(message);      // Store the message on chain
            Ok(())
        }
    }
}
```

### 📝 Explanation (Line-by-Line)

- **Imports**: FRAME macros and traits needed to define pallet logic  
- **`#[pallet]`**: Marks the module as a Substrate pallet  
- **Config**: Defines pallet dependencies (none in this simple case)  
- **Storage**: `StoredMessage` saves a message of type `Vec<u8>`  
- **Function**: `set_message()` stores the message on-chain when called  

### ✅ What Happens?

- A user calls `set_message("Hello Polkadot")`
- The message gets stored in the blockchain's state
- Anyone can later query `stored_message` to retrieve it

---

## 🔗 Turning It Into a Parachain

After building your blockchain:

1. Use [Cumulus](https://github.com/paritytech/cumulus) to make it parachain-compatible  
2. Register it on the Polkadot Relay Chain  
3. Use XCMP (Cross-Chain Message Passing) to communicate with other chains  

---

## 🧠 Core Concepts in Substrate

### 1. FRAME

**Framework for Runtime Aggregation of Modularized Entities**  
It powers your blockchain logic using libraries and macros.

### 📦 Key Components

#### 🔹 pallet

A reusable module or feature in your blockchain (e.g., balances, identity, voting).

Each pallet defines:

- Its own storage
- Events
- Errors
- Callable functions (extrinsics)

Example:

```rust
#[pallet::storage]
pub type StoredValue<T> = StorageValue<_, u32>;
```

#### 🔹 frame_support

Gives you helper macros/tools to build pallets.

Example:

```rust
use frame_support::{pallet_prelude::*, dispatch::DispatchResult};
```

#### 🔹 frame_system

Gives access to low-level blockchain info: who is sending a transaction, block numbers, etc.

Example:

```rust
let sender = ensure_signed(origin)?;
```

#### 🔹 Storage Types

- `StorageValue<_, u32>`: single value  
- `StorageMap<_, u32, u64>`: key-value map  
- `StorageDoubleMap<_, u32, u32, u64>`: double-key map  

#### 🔹 DispatchResult

Return type for public functions, used to signify success or failure.

Example:

```rust
pub fn do_something() -> DispatchResult {
    Ok(())
}
```

#### 🔹 OriginFor<T>

Identifies who called the function. Used with `ensure_signed(origin)`.

#### 🔹 Vec<u8>

A byte vector, used for strings or arbitrary data.

Example:

```rust
message: Vec<u8> = b"Hello World".to_vec();
```

#### 🔹 Events & Errors

- **Events**: Notify what happened (e.g., "user stored a message")  
- **Errors**: Handle failures clearly  

Example event:

```rust
#[pallet::event]
pub enum Event<T> {
    MessageStored(T::AccountId, Vec<u8>),
}
```

---

## 🔁 Putting It All Together

1. User sends a transaction  
2. `frame_system` authenticates the user  
3. Your custom pallet runs the logic (e.g., store message)  
4. `frame_support` makes it easier with macros  
5. Result: success emits event, failure returns an error  

---

### ✅ You’ve Just Built a Simple Web3 Blockchain with Substrate!

Feel free to extend this with more features, connect to a relay chain, and explore the Polkadot ecosystem!
