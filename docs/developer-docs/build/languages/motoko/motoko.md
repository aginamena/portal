# Motoko Programming Language

:::tip

The Motoko programming language continues to evolve with each release of the DFINITY Canister SDK and with ongoing updates to the Motoko compiler. Check back regularly to try new features and see what’s changed.

:::

The Motoko programming language is a new, modern and type safe language for developers who want to build the next generation of distributed applications to run on the Internet Computer blockchain network. Motoko is specifically designed to support the unique features of the Internet Computer and to provide a familiar yet robust programming environment. As a new language, Motoko is constantly evolving with support for new features and other improvements.

The Motoko compiler, documentation and other tooling is [open source](https://github.com/dfinity/motoko) and released under the Apache 2.0 license. Contributions are welcome.

## Native canister smart contract support

Motoko has native support for Internet Computer canister smart contracts.

A canister smart contract (or canister for short) is expressed as a Motoko actor. An actor is an autonomous object that fully encapsulates its state and communicates with other actors only through asynchronous messages.

``` motoko name=counter
actor Counter {

  var value = 0;

  public func inc() : async Nat {
    value += 1;
    return value;
  };
}
```

``` motoko name=counter file =./Counter.mo
```

## Code sequentially in direct style

On the Internet Computer, canisters can communicate with other canisters by sending asynchronous messages.

Asynchronous programming is hard, so Motoko enables you to author asynchronous code in much simpler, sequential style. Asynchronous messages are function calls that return a *future*, and the `await` construct allows you to suspend execution until a future has completed. This simple feature avoids the "callback hell" of explicit asynchronous programming in other languages.

``` motoko include=counter
actor Factorial {

  var last = 1;

  public func next() : async Nat {
    last *= await Counter.inc();
    return last;
  }
};

ignore await Factorial.next();
ignore await Factorial.next();
await Factorial.next();
```

## Modern type system

Motoko has been designed to be intuitive to those familiar with JavaScript and other popular languages, but offers modern features such as sound structural types, generics, variant types, and statically checked pattern matching.

``` motoko
type Tree<T> = {
  #leaf : T;
  #branch : {left : Tree<T>; right : Tree<T>};
};

func iterTree<T>(tree : Tree<T>, f : T -> ()) {
  switch (tree) {
    case (#leaf(x)) { f(x) };
    case (#branch{left; right}) {
      iterTree(left, f);
      iterTree(right, f);
    };
  }
};

// Compute the sum of all leaf nodes in a tree
let tree = #branch { left = #leaf 1; right = #leaf 2 };
var sum = 0;
iterTree<Nat>(tree, func (leaf) { sum += leaf });
sum
```

## Autogenerated IDL files

A Motoko actor always presents a typed interface to its clients as a suite of named functions with argument and (future) result types.

The Motoko compiler (and SDK) can emit this interface in a language neutral format called Candid, so other canisters, browser resident code and smart phone apps that support Candid can use the actor’s services. The Motoko compiler can consume and produce Candid files, allowing Motoko to seamlessly interact with canisters implemented in other programming languages (provided they support Candid).

For example, the previous Motoko `Counter` actor has the following Candid interface:

``` candid
service Counter : {
  inc : () -> (nat);
}
```

## Orthogonal persistence

The Internet Computer persists the memory and other state of your canister as it executes. Thus the state of a Motoko actor, including its in-memory data structures, survive indefinitely. Actor state does not need to be explicitly "restored" and "saved" to external storage, with every message.

For example, in the following `Registry` actor (canister), that assigns sequential IDs to textual names, the state of the hash table is preserved across calls, even though the state of the actor is replicated across many Internet Computer node machines, and typically not resident in memory.

``` motoko
import Text "mo:base/Text";
import Map "mo:base/HashMap";

actor Registry {

  let map = Map.HashMap<Text, Nat>(10, Text.equal, Text.hash);

  public func register(name : Text) : async () {
    switch (map.get(name)) {
      case null {
        map.put(name, map.size());
      };
      case (?id) { };
    }
  };

  public func lookup(name : Text) : async ?Nat {
    map.get(name);
  };
};

await Registry.register("hello");
(await Registry.lookup("hello"), await Registry.lookup("world"))
```

## Upgrades

Motoko provides numerous features to help you leverage orthogonal persistence, including language features that allow you to retain a canister’s data as you upgrade the code of the canister.

For example, Motoko lets you declare certain variables as `stable`. The values of `stable` variables are automatically preserved across canister upgrades.

Consider a stable counter:

``` motoko
actor Counter {

  stable var value = 0;

  public func inc() : async Nat {
    value += 1;
    return value;
  };
}
```

It can be installed, incremented *n* times, and then upgraded, without interruption, to, for example, the richer implementation:

``` motoko
actor Counter {

  stable var value = 0;

  public func inc() : async Nat {
    value += 1;
    return value;
  };

  public func reset() : async () {
    value := 0;
  }
}
```

Because `value` was declared `stable`, the current state, *n*, of the service is retained after the upgrade. Counting will continue from *n*, not restart from `0`.

Because the new interface is compatible with the previous one, existing clients referencing the canister will continue to work, but new clients will be able to exploit its upgraded functionality (the additional `reset` function).

For scenarios that can’t be solved using stable variables alone, Motoko provides user-definable upgrade hooks that run immediately before and after upgrade, and allow you to migrate arbitrary state to stable variables.

## And more …​

Motoko provides many other developer productivity features, including subtyping, arbitrary precision arithmetic and garbage collection.

Motoko is not, and is not intended to be, the only language for implementing canister smart contracts. If it doesn’t suit your needs, there is a canister development kit (CDK) for the Rust programming language. Our goal is to enable any language (with a compiler that targets WebAssembly) to be able to produce canister smart contracts that run on the Internet Computer and interoperate with other, perhaps foreign, canister smart contracts through language neutral Candid interfaces.

Its tailored design means Motoko should be the easiest and safest language for coding on the Internet Computer, at least for the forseeable future.