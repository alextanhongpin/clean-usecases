class: center, middle

# PIPE Architecture

---

# Agenda

1. Introduction
2. Deep-dive
3. ...

---

# Introduction

Unix pipe

```bash
ls /bin /usr/bin | sort | uniq | tee abc.txt | grep out
```

- composition of operation
- chaining by taking input, produce output and passing it to to next operation

---

# How is this different from ...

- Clean architecture, hexagonal architecture, onion, etc. focuses on separation of layers. 
- Business layer is just another layer, lack of solid implementation
- DDD focuses on the other extreme - business logic belongs to domain models, with aggregate, value objects, repositories, bounded context etc representing again more layers. Tries to separate business logic from external dependency
- In DDD, domain models/aggregate are then called in usecase layer/application service. It takes an expert to design a really solid (no pun, and not related to SOLID) domain layer

--- 

# Problem 

- [ ] large usecase function (few hundred lines) with multiple steps
- [ ] multiple service calls, different entities and multiple interactions. Domain models rarely work alone. If they do, that is a sign you are building CRUD. 
- resulting in poor redability
- difficulty in debugging immediate state, manual log, not testable
- no clear separation between IO
- not much is said about business usecase layer
- DDD and clean architecture focuses more on defining boundaries and logical separation, usecase layer is just treated as one layer
- DDD concepts are confusing enough, the more you know, your achitecture takes toll on the transformation
- no two developers share the same understanding
- stick to the basics, procedural with template pattern

---

# What we are looking for

- a way to translate requirements to usecases
- keeping logic reusable
- separate data from behaviour (ddd focuses on putting them in domain models, however in practice, we may want to separate them)
- a lot of ddd concepts may not apply when there are more complex orchestration needed, e,g generate CSV from request, saga long living transactions, state machine transition in a model, send an email/mobile notification on certain events, process webhooks, checking permission or role. They are certainly IO and view related, as well as authorization processes.


---

## Why Pipe?

- without piping, functions needs to be nested
- gets messy
```go
foo(bar(baz(new_function(other_function()))))
```
---

Elixir:
```elixir
other_function() |> new_function() |> baz() |> bar() |> foo()
```

---

JS promise chain:

```js
doSomething()
.then(result => doSomethingElse(result))
.then(newResult => doThirdThing(newResult))
.then(finalResult => {
  console.log(`Got the final result: ${finalResult}`);
})
.catch(failureCallback);
```

---

## Disadvantage

- intermediate state is not stored
- hard to pass dependencies
- not all language support it (or do they?)

---

## PIPE in Golang

## Advantages

- cleaner control flow
- predictable intermediate state
- fulfils interface


---

# Template pattern


- defines a series of procedural process
- hides the implementation
- allows strategy pattern
- plug and play business logic



