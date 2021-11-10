class: center, middle

# PIPE Architecture

---

# Agenda

1. Introduction
2. Deep-dive
3. ...

---

# Introduction

Linux pipe

```bash
ls /bin /usr/bin | sort | uniq | tee abc.txt | grep out
```

often praised for simplicity and doing one thing (SRP)


# Problem 

- [ ] large usecase function (few hundred lines)
- [ ] multiple service calls, different entities and multiple interactions
- resulting in poor redability
- difficulty in debugging immediate state, manual log, not testable
- no clear separation between IO
- not much is said about business usecase layer
- DDD and clean architecture focuses more on defining boundaries and logical separation, usecase layer is just treated as one layer
- DDD concepts are confusing enough, the more you know, your achitecture takes toll on the transformation
- no two developers share the same understanding
- stick to the basics, procedural with template pattern



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
