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

## What is Cumulus?
Cumulus is a set of tools and libraries that helps you turn your Substrate-based blockchain into a parachain that can connect and operate within the Polkadot or Kusama networks.

### Why is Cumulus important?
Polkadot’s architecture is made up of a Relay Chain (the main chain) and multiple Parachains (independent blockchains running in parallel).

- To become a parachain, your blockchain needs to be compatible with the Relay Chain’s rules and protocols.
- Cumulus makes this possible by providing:
  - A lightweight client implementation of the Relay Chain interface,
  - Tools to integrate your Substrate node with the Relay Chain,
  - Support for cross-chain communication (XCMP) and consensus coordination.


## Analogy:
Imagine the Polkadot network as an airport (Relay Chain) with many airplanes (parachains). Each airplane flies independently but needs to follow airport rules for landing, takeoff, and communication.

Cumulus is like the training and certification that makes your airplane compliant with the airport so it can safely land and take off, communicate with control towers, and share airspace.

### In short:
Cumulus = Glue that connects your Substrate blockchain as a parachain on Polkadot.

Enables your chain to benefit from Polkadot’s shared security and interoperability features.

---

## 🧠 Core Concepts in Substrate (With Analogies)

### 1. FRAME

**Framework for Runtime Aggregation of Modularized Entities**

Think of FRAME as a **building kit or Lego set** for your blockchain. It provides reusable bricks (called pallets) that you can assemble to build your blockchain’s logic — like building a house or a car out of Lego blocks instead of making everything from scratch.

---

### 📦 Key Components

#### 🔹 pallet

A pallet is like a **Lego block or module** with a specific feature or function — such as handling balances, user identities, or voting mechanisms.

- Each pallet has its **own storage space** (like a box where it keeps its data),
- Can generate **events** (like signals to tell you something happened),
- Can define **errors** (to handle when things go wrong),
- And provides **functions** (actions users can perform, called extrinsics).

> **Analogy:** If your blockchain was a car, pallets would be the engine, the brakes, the seats, and the steering wheel — all separate modules working together.

Example:

```rust
#[pallet::storage]
pub type StoredValue<T> = StorageValue<_, u32>;
```

This declares a storage box inside the pallet to hold a single number.

---

#### 🔹 frame_support

This is the **toolbox or helper kit** that gives you useful macros and utilities to build your pallets faster and cleaner.

> **Analogy:** Like having a screwdriver, hammer, and measuring tape in your toolkit so you don’t have to make those tools yourself.

Example:

```rust
use frame_support::{pallet_prelude::*, dispatch::DispatchResult};
```

---

#### 🔹 frame_system

This pallet acts like the **operating system of your blockchain**. It manages basic tasks like knowing who sent a transaction, when a block was created, and controlling access.

> **Analogy:** If the blockchain is a factory, `frame_system` is the factory manager who keeps track of workers (users) and time (blocks).

Example:

```rust
let sender = ensure_signed(origin)?;
```

This checks and extracts the user who signed (sent) a transaction.

---

#### 🔹 Storage Types

Think of storage types as **different types of containers** to keep your data safe in the blockchain’s warehouse:

- `StorageValue<_, u32>`: A **single box** holding one item (e.g., a number).
- `StorageMap<_, u32, u64>`: A **labeled shelf** where each label (key) maps to an item (value).
- `StorageDoubleMap<_, u32, u32, u64>`: Like a **two-keyed filing cabinet**, needing two keys to find the stored item.

---

#### 🔹 DispatchResult

This is like a **status flag** telling whether a function succeeded or failed.

> **Analogy:** After you flip a light switch, you want to know if the light turned on (success) or if the bulb is broken (failure).

Example:

```rust
pub fn do_something() -> DispatchResult {
    Ok(())
}
```

---

#### 🔹 OriginFor<T>

This tells you **who is calling the function** — it helps verify the identity of the transaction sender.

> **Analogy:** Like showing your ID before entering a restricted room.

Used with `ensure_signed(origin)` to confirm the caller is a real user.

---

#### 🔹 Vec<u8>

This is a **vector (list) of bytes**, commonly used to store text or arbitrary data.

> **Analogy:** Imagine a string of letters written in a secret code (bytes). For example, `"Hello"` is stored as a sequence of byte numbers.

Example:

```rust
message: Vec<u8> = b"Hello World".to_vec();
```

---

#### 🔹 Events & Errors

- **Events** are **notifications or signals** emitted when something important happens.

  > **Analogy:** Like a doorbell ringing to alert someone.

- **Errors** tell you if something went wrong and why.

  > **Analogy:** A red warning light on your car’s dashboard.

Example event:

```rust
#[pallet::event]
pub enum Event<T> {
    MessageStored(T::AccountId, Vec<u8>),
}
```

This event tells the system that a user stored a message.


---

## 🔁 Putting It All Together

1. User sends a transaction  
2. `frame_system` authenticates the user  
3. Your custom pallet runs the logic (e.g., store message)  
4. `frame_support` makes it easier with macros  
5. Result: success emits event, failure returns an error  

---


## How to Test Your Substrate Blockchain
1. Unit Testing
- Write Rust unit tests inside your pallets.
- Use Rust’s built-in testing framework (#[test]) to test individual functions and logic.
- Run tests with

```bash
cargo test
```

2. Using Substrate Front-End Template
- Connect your node to the Substrate Front-End Template.
- It’s a React-based UI where you can manually interact with your blockchain (submit transactions, query storage).
- Great for manual functional testing.

3. Command-Line Tools
- Substrate Node CLI
Run your node locally and interact via the command line.

- Polkadot.js Apps
Web-based UI to interact with any Substrate-based chain.
Access at: https://polkadot.js.org/apps


### ✅ You’ve Just Built a Simple Web3 Blockchain with Substrate!

