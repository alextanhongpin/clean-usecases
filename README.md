# clean-usecases
Programming with abstraction

I've tried 

- domain-driven design
- clean/hexagonal/onion architecture

However, they introduce a lot of complexity (concepts, as well as layers) that is hard to understand, has many different interpretations, hard to reason about, and most importantly, hard to ignore, leaving people with FUD (fear, uncertainty and doubt).

Sometimes, we just want to get the job done in a way that is easy to understand.


## Principles

Clean usecases is all about control flow - what are the steps that is required to run the usecase to completion.

It does not concern itself with
- domain model attributes. Unlike DDD, where you create domain models and define attributes, clean usecase only focus on behaviours. Where the data is from is not important.
- persistence or I/O. Each step is mainly focusing on behaviour/business logic. Data could be injected externally.
- implementation details. Each step could be done through composition of different functions, and different flows could share the same step.
- layers. How you define your layer is up to you. 

## Sample

```go
package main

import (
	"errors"
	"fmt"
)

func main() {
	auth := &authenticateImpl{email: "john doe"}
	if err := Authenticate(auth); err != nil {
		panic(err)
	}
	fmt.Println("Hello, playground")
}

type PipeFn func() error

func Pipe(fns ...PipeFn) error {
	for _, fn := range fns {
		if err := fn(); err != nil {
			return err
		}
	}
	return nil
}

func Authenticate(flow interface {
	ValidateEmail() error
	ValidatePassword() error
	EncryptPassword() error
}) error {

	// Race(flow.doSth, flow.doSthElse)
	// Retry(flow.failsRandomly)
	return Pipe(
		flow.ValidateEmail,
		flow.ValidatePassword,
		flow.EncryptPassword,
	)
}

type authenticateImpl struct {
	email    string
	password string

  // Pipe must produce an output
	output struct {
		encryptedPassword string
	}
}

func (i *authenticateImpl) ValidateEmail() error {
	if len(i.email) == 0 {
		return errors.New("email is required")
	}
	fmt.Println("validating email")
	return nil
}

func (i *authenticateImpl) ValidatePassword() error {
	fmt.Println("validating password")
	return nil
}

func (i *authenticateImpl) EncryptPassword() error {
	fmt.Println("encrypting password password")
	return nil
}
```
