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

## Why Pipe?

- without piping, functions needs to be nested
- gets messy
```go
foo(bar(baz(new_function(other_function()))))
```

Elixir:
```elixir
other_function() |> new_function() |> baz() |> bar() |> foo()
```

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

## Disadvantage

- intermediate state is not stored
- hard to pass dependencies
- not all language support it (or do they?)

## PIPE in Golang

## Advantages

- cleaner control flow
- predictable intermediate state
