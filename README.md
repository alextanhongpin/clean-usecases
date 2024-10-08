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
- layers. How you define your layer is up to you. In DDD, domain layer (entity, service, repository, entity, value object etc) is the main abstraction, in clean usecase, usecase layer becomes the main abstraction. We do not care about the details on each entity, we only care how they operate as a whole


It is
- pure abstraction
- focuses on control flow
- based on piping data and transforming them from one state to another
- implementation should be fully testable without depending on persistence or I/O
- when doing BDD (behaviour driven development), you can initialize the implementation to fulfill the `Given` scenario, so you only tests the `When` and `Then`. E.g. based on the sample below, we can write a test where 

```
Given a user with valid email and password, 
When authenticating the user
Then the password should be encrypted
```

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
	// Run parallel, behind the scenes uses errgroup.Do
	// Parallel(flow.DoSth)
	
	// First to execute wins
	// Race(flow.doSth, flow.doSthElse)
	
	// Allow retries 
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

## Observability

We can improve the pipe by 
- logging the number of completed steps
- logging the function name that fails

```go
package main

import (
	"errors"
	"fmt"
	"reflect"
	"runtime"
)

func main() {
	fmt.Println(Pipe(ValidateName))
	fmt.Println()
	
	v := &validator{err: errors.New("invalid age")}
	fmt.Println(Pipe(v.ValidateAge))
	CreateUser(v)
	fmt.Println()
	
	v = &validator{}
	fmt.Println(Pipe(v.ValidateAge, ValidateName))
	fmt.Println()
	
	CreateUser(v)
}

func CreateUser(flow interface {
	ValidateAge() error
}) error {
	n, err := Pipe(flow.ValidateAge)
	fmt.Println(n)
	return err
}

func ValidateName() error { return errors.New("bad request") }

type validator struct{ err error }

func (v *validator) ValidateAge() error { return v.err }

type PipeFn func() error

func Pipe(fns ...PipeFn) (int, error) {
	var count int
	for _, fn := range fns {
	// NOTE: Wrap the error in a pipe error with the steps and function name 
		if err := fn(); err != nil {
			fmt.Println(GetFunctionName(fn))
			return count, err
		}
		count++
	}
	return count, nil
}

func GetFunctionName(i interface{}) string {
	return runtime.FuncForPC(reflect.ValueOf(i).Pointer()).Name()
}
```

## Intermediate states

- Intermediate states can be kept in `state` struct
- Makes testing easier, since we can now debug each step/test each step individually
- No longer a black box function with thousand lines, each step can report their own state

## Testing

- mocking and stubbing is much easier

## Side-effects and Lazy-loading

For fetching data lazily
- side effects cannot be avoided, we still need them (can be data from external HTTP request or db)
- however, there's no mutation involves, so they are only done outside of the flow
- only queries are possible


## How can we write leaner code?

- reduce number of layers, packages, structs and type
- abstraction over implementation
- separate infrastructure concerns from business logic

what layers we need
- presentation
- usecase
- infrastructure (including repository)


what layers to skip?
- domain (?)
- https://www.waitingforcode.com/general-big-data/acid-2-0/read
https://www.alibabacloud.com/blog/an-alibaba-cloud-technical-experts-insight-into-domain-driven-design-domain-primitive_596116
https://www.alibabacloud.com/blog/how-to-code-complex-applications-core-java-technology-and-architecture_595506

## Errors vs boolean

Sometimes we just want to terminate the flow early, but not using the error.
- use a sentinel error, e.g, ErrNoop to terminate early, but not handling this error later
- boolean output could be stored and used for a more custom control flow
- use pointer boolean instead to indicate everything is set
- use constructors to ensure all input is set

## With Context and Message Passing state

```go
// You can edit this code!
// Click here and start typing.
package main

import (
	"context"
	"errors"
	"fmt"
)

func main() {
	uc := new(RegisterUsecase)
	uc.repo = &mockRegisterRepo{}
	uc.uow = &mockUnitOfWork{}
	ctx := context.Background()
	req := RegisterDto{Email: "john.appleseed@mail.com", Password: "123"}
	fmt.Println(uc.Do(ctx, req))
	fmt.Println("Hello, 世界")
}

type CreateUserRequest struct {
	Email             string
	EncryptedPassword string
}

type User struct {
	Name string
}

type registerRepository interface {
	CreateUser(ctx context.Context, params CreateUserRequest) (*User, error)
}

type RegisterUsecase struct {
	repo registerRepository
	uow  unitOfWork
}

type State struct {
	In                RegisterDto
	EncryptedPassword string
	User              result[*User]
}

type result[T any] struct {
	data T
	err  error
}

func makeResult[T any](v T, err error) result[T] {
	return result[T]{
		data: v,
		err:  err,
	}
}

func (r *result[T]) Unwrap() (T, error) {
	return r.data, r.err
}

type RegisterDto struct {
	Email    string
	Password string
}

func (dto *RegisterDto) Valid() error {
	if dto.Email == "" || dto.Password == "" {
		return errors.New("required")
	}
	return nil
}

func Tx[T *V, V any](ctx context.Context, uow unitOfWork, state T, steps ...Step[T, V]) Step[T, V] {
	return func(ctx context.Context, state T) error {
		return uow.RunInTx(ctx, func(ctx context.Context) error {
			return Exec(ctx, state, steps...)
		})
	}
}

func (uc *RegisterUsecase) Do(ctx context.Context, in RegisterDto) (*User, error) {
	if err := in.Valid(); err != nil {
		return nil, err
	}
	state := &State{In: in}
	if err := Exec(ctx, state,
		uc.EncryptPassword,
		uc.CreateUser,
		Tx(ctx, uc.uow, state,
			uc.EncryptPassword,
			uc.CreateUser,
		),
	); err != nil {
		return nil, err
	}

	return state.User.Unwrap()
}

func (uc *RegisterUsecase) EncryptPassword(ctx context.Context, state *State) error {
	fmt.Println("encrypt password")
	state.EncryptedPassword = "encrypted:" + state.In.Password
	return nil
}

func (uc *RegisterUsecase) CreateUser(ctx context.Context, state *State) error {
	fmt.Println("")
	state.User = makeResult(uc.repo.CreateUser(ctx, CreateUserRequest{
		Email:             state.In.Email,
		EncryptedPassword: state.EncryptedPassword,
	}))
	return state.User.err
}

type Step[T *V, V any] func(ctx context.Context, state T) error

func Exec[T *V, V any](ctx context.Context, state T, steps ...Step[T, V]) error {
	for _, step := range steps {
		if err := step(ctx, state); err != nil {
			return err
		}
	}
	return nil
}

func Chain[T *V, V any](ctx context.Context, state T, steps ...Step[T, V]) Step[T, V] {
	return func(ctx context.Context, state T) error {
		for _, s := range steps {
			if err := s(ctx, state); err != nil {
				return err
			}
		}

		return nil
	}
}

type unitOfWork interface {
	RunInTx(ctx context.Context, fn func(ctx context.Context) error) error
}

type mockRegisterRepo struct{}

func (r *mockRegisterRepo) CreateUser(ctx context.Context, params CreateUserRequest) (*User, error) {
	return &User{
		Name: "john",
	}, nil
}

type mockUnitOfWork struct{}

func (uow *mockUnitOfWork) RunInTx(ctx context.Context, fn func(context.Context) error) error {
	return fn(ctx)
}

```


## When to use it

Never. Although a lot of things can be anstracted, the control flow probably shouldn't.

Also, it limits the way we write code, and stuff like defer becomes hard to add.
