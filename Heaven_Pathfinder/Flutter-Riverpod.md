## What it is?
Riverpod is a [[Flutter]] package designed for widget-state management. Basically to update interdependant components in several widgets at once without pain in ass.

## How to integrate?
In the pubspec.yaml file in the flutter project, in the dependencies the `flutter_riverpod: *version*` needs to be pasted

## Core essense of the solution
As this libraryis designed to exchange the information, there are two groups of instruments: Providers and consumers
### What is a Riverpod Provider?
The Riverpod documentation defines a Provider as **an object that encapsulates a piece of state and allows listening to that state**.

With Riverpod, providers are the core of everything:
- They completely replace design patterns such as **[singletons](https://codewithandrea.com/articles/flutter-singletons/)**, **service locators**, **dependency injection**, and **InheritedWidgets**.(Dunno what the phouque these are, maybe will add info later) [[Garbage collector]]
- They allow you to store some state and easily access it in multiple locations.
- They allow you to optimize performance by filtering widget rebuilds or caching expensive state computations.
- They make your code more testable, since each provider can be overridden to behave differently during a test.
### Creating and Reading a Provider
Let's get started by creating a basic "Hello world" provider:

```
// provider that returns a string value
final helloWorldProvider = Provider<String>((ref) {
  return 'Hello world';
});
```

This is made of three things:
1. **The declaration**: `final helloWorldProvider` is the global variable that we will use to read the state of the provider
2. **The provider**: `Provider<String>` tells us what **kind** of provider we're using (more on this below), and the **type** of the state it holds.
3. **A function** that creates the state. This gives us a `ref` parameter that we can use to read other providers, perform some custom dispose logic, and more.

## Creating and Reading a Provider
### 1. Using a ConsumerWidget
### 2. Using a Consumer
### 3. Using ConsumerStatefulWidget & ConsumerState

## What is a WidgetRef?

## Eight different kinds of providers
### 1. Provider
### 2. StateProvider
### 3. StateNotifierProvider
### 4. FutureProvider
### 5. StreamProvider
### 6. ChangeNotifierProvider
### New in Riverpod 2.0: NotifierProvider and AsyncNotifierProvider

## When to use ref.watch vs ref.read?
### Listening to Provider State Changes

## Additional Riverpod Features

## The autoDispose modifier
### Caching with Timeout

## The family modifier

## Dependency Overrides with Riverpod

## Combining Providers with Riverpod
### Passing Ref as an argument

## Scoping Providers

## Filtering widget rebuilds with "select"

## Testing with Riverpod
### How to mock and override dependencies in tests
### Where to learn more about Testing with Riverpod?

## Logging with ProviderObserver

## Quick Note about App Architecture with Riverpod
