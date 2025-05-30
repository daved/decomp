# Decomp: Modular, Maintainable, Testable Code

The most common primitive used to express behavior is the function, yet we do not have nomenclature for decomposing functions when they grow unwieldy, whether that be technically or semantically. It may seem unimportant to dig into the whys of functions and data structures. However, we name complex patterns like Singleton and Bridge. We also formalize paradigms like Functional Programming with rigorous math. This easily missed void is one we can, and should, address. This document is intended to kick-off a common language for code decomposition with the hope of realizing a sharper engineering mindset in software development.

Decomposing code involves breaking down behavior and data into the most relevant and purposeful elements. Composing code involves integrating those well-decomposed elements minimally and effectively. The following "categories of purpose" provide practical, adaptable guidelines through clear, structured naming. They cover arranging behavior and data, conditioning desired control flows with error handling, and testing source code.

## Arranging Behavior and Data: Categories of Purpose

Here we cover how functions and types (i.e. behavior and data) can be broken down to their smallest elements.

### Functionalization

- Formation of Values
    - Construction   (constructor, calculator, getter [dynamic value])
    - Articulation   (closure)
    - Modification   (setter)
- Relation of Behavior
    - Exposition     (handler)
    - Junction       (multiplexer)
    - Seclusion      (subhandler)
- Notation of Attributes
    - Identification (discriminator [no/irrelevant value])
    - Elucidation    (getter [static value])

#### Formation of Values: Examples

##### Construction

Constructions construct values whether that be by instantiation, calculation, or inspection/derivation.

###### Constructor

```go
type App struct {
	Name string
}

func NewApp(name string) *App { // instantiation
	return &App{Name: name}
}
```

###### Calculator

```go
func add(a, b int) int { // calculation
	return a+b
}
```

###### Getter

```go
type App struct {
	Name string
}

func Name(a *App) string { // inspection
	return a.Name
}

func Greeting(a *App) string { // derivation
	name := a.Name
	if name == "" {
		name = "App"
	}
	return "Hello from " + name
}
```

##### Articulation

Articulations construct scopes that are expanded (inclusion) and/or make a "change in direction" in the control flow (diversion).

###### Closure

```go
func Accumulation(initVal int) func(addend int) int { // instantiation
	return func(a int) int { // diversion with inclusion
		initVal += a
		return initVal
	}
}
```

##### Modification

Modifications apply a mutation to some value (whether apparent or actual).

###### Setter

```go
type App struct {
	name  string
	start int
}

func AppSetName(a App, name string) App { // apparent mutation (copy-based)
	a.name = name
	return a
}

func SetAppName(a *App, name string) { // actual mutation (reference-based)
	a.name = name
}
```

#### Relation of Behavior: Examples

##### Exposition

Expositions convey behaviors through invocation of other code.

###### Handler

```go
func HandleHello(w http.ResponseWriter, r *http.Request) { // invocation
	if !isAuthorized(r) {
		http.Error(w, "Unauthorized", http.StatusUnauthorized)
		return
	}
	name := nameFromRequest(r)
	fmt.Fprintf(w, "Hello, %s!", name)
}
```

##### Junction

Junctions inspect one ore more values in order to determine control flow resolution.

###### Multiplexer

```go
func RouteHandling(data []byte) error { // resolution
	switch {
	case bytes.HasPrefix(data, []byte{0xAA, 0xCC}):
		return handleAlpha(data)
	case bytes.HasPrefix(data, []byte{0xBB, 0xCC}):
		return handleBeta(data)
	default:
		return fmt.Errorf("Unknown data prefix: %x", data[:min(2, len(data))])
	}
}
```

##### Seclusion

Seclusions group invocational code into smaller pieces for the sake of organization. This may be for cleanliness of calling code, testability, reusability, etc.

###### Subhandler

```go
func HandleHello(w http.ResponseWriter, r *http.Request) { // handler performing its invocation(s)
	// other behavior ...
	PrintHello(w, r)
}

func PrintHello(w http.ResponseWriter, r *http.Request) { // isolation
	name := nameFromRequest(r)
	fmt.Fprintf(w, "Hello, %s!", name)
}
```

#### Notation of Attributes: Examples

##### Identification

Identifcations help shape types for differentiation as a form of metadata and usually have no interesting behavior.

###### Discriminator

```go
type ExitError struct {
	Code int
}

// other methods...

func (e *ExitError) Silent() {} // presence permits differentiation
```

##### Elucidation

Elucidations offer values (typically static) for clarification as a form of metadata and usually have no interesting behavior.

###### Getter

```go
type ExitError struct {
	Code int
}

// other methods...

func (e *ExitError) IsSilent() bool { // value permits clarification
	return false
}
```

### Structuralization

- Formalization of Data
    - Inscription       (direct model)
    - Stylization       (abstract model)
- Association of Types
    - Contextualization (dependencies)

#### Formalization of Data: Examples

##### Inscription

Inscriptions are types that are minimally abstracted from their fundamental information. Behaviors associated with them tend to be restricted to transmission/translation (e.g. serialization).

###### Direct Model

```go
type ConfigFile struct {
	DBHost string
	DBPort int
	DBUser string
	DBPass string
}
```

##### Stylization

Stylizations are types that are abstracted from their fundamental information. Behaviors associated with them tend to be based on developer preference/convenience (i.e. application needs).

###### Abstract Model

```go
type DBDetails struct {
	Driver string
	DSN    string
}

func NewDBDetails(host string, port int, user, pass string) *DBDetails {
	return &DBDetails{
		Driver: "postgres",
		DSN: fmt.Sprintf("host=%s port=%d user=%s password=%s", host, port, user, pass),
	}
}
```

#### Association of Types: Examples

##### Contextualization

Contextualizations focus solely on application organization. Specifically, these types carry dependencies down into the application's deeper layers so that more magical solutions like globals or dependency injection containers are not necessary.

###### Dependencies

```go
type App struct {
	db *sql.DB
}

func NewApp(db *sql.DB) *App {
	return &App{
		db: db,
	}
}

func (a *App) PingDB() error {
	return a.db.Ping()
}
```

## Conditioning

Here we cover how values can be used to indicate when our desired control flow must be turned from so that some error condition can be handled.

### Typification

- Pointer                  (nil/null)
- Enums                    (token)
- Bit-shifted Integers     (rwx)
- Special Meaning Integers (-1)

#### Examples

##### Pointer

```go
func main() {
	a := app.New(creds)
	if a == nil {
		log.Fatalln("app was not constructed properly")
	}
	a.Run()
}
```

##### Enums

```go
func main() {
	status := app.Run(creds)
	if status == app.Error || status == app.Unknown {
		log.Fatalf("app was not constructed: %s\n", status)
	}
```

##### Bit-shifted Integers

```go
const (
	SuccessOne = 1 << iota // 1 (0b01)
	SuccessTwo             // 2 (0b10)
)

func main() {
	result := run(creds)
	if result&SuccessOne != 0 {
		fmt.Println("One succeeded")
	} else {
		fmt.Println("Failed to complete One")
	}
	if result&SuccessTwo != 0 {
		fmt.Println("Two succeeded")
	} else {
		fmt.Println("Failed to complete Two")
	}
}
```

##### Special Meaning Integers

```go
func main() {
	i := strings.Index("hello", "x")
	fmt.Println(i)                   // Output: -1
}
```

### Disjunction

- Error (XOR Value)
    - Token (simple, "is")
    - Behavior (complex, "as")

#### Error (XOR Value): Examples

##### Token (simple, "is")

```go
func main() {
	r := strings.NewReader("")
	b := make([]byte, 1)
	_, err := r.Read(b)
	if errors.Is(err, io.EOF) {
		fmt.Println("End of input")
	}
}
```

##### Behavior (complex, "as")

```go
func main() {
	_, err := net.Dial("tcp", "invalid:host")
	var opErr *net.OpError
	if errors.As(err, &opErr) {
		fmt.Println("Network operation failed:", opErr.Op)
	}
}
```

### Resignation

- Panic
- Abrupt return

#### Examples

##### Panic

```go
func divideBy(n, d int) int {
	if d == 0 {
		panic("division by zero")
	}
	return n / d
}
```

##### Abrupt return

```go
func greet(name string) {
	if name == "" {
		return
	}
	fmt.Printf("Hello, %s\n", name)
}
```

## Testing

Here we cover how tests can be bound by relationships, scope, and expectations associated with the targeted functions and types.

### Scopes

- Unit    (function)
- Surface (API - object, package/module, etc.)
- System  (system/artifact)

#### Unit Testing

Ideally, unit testing should be limited to highly focused pure functions. Pure functions are functions that have no side-effects (i.e. no external state is changed), and always return the same output for the same input. However, for the sake of pragmatism, rather than disqualifying a test from being a unit test because it is not "ideal", it is helpful to prioritize the intention of a function and its testing. Using this lens, we can then simply acknowledge deviations for the sake of communication and comprehension and avoid unprofitable arguments about technical boundaries. 

#### Surface Testing

Expanding out from unit testing, surface tests begin to embrace the integration of multiple types, functions, and/or primitives. This happens naturally due to how we engage with more interesting "shapes" like object API surfaces, package/module API surfaces, and substantially complex functions. This is especially true as our perspective shifts from "maintainer who holds secret knowledge" of some portion of code and towards "user who is limited to the docs".

#### System Testing

Expanding, again, from surface testing, system tests bring us nearly fully to an external perspective. These tests tend to be established by product/user/customer expectations rather than those of internal developers. To a certain degree, this is where "nothing is sacred" and no secret knowledge should be allowed. This is as close to the real world as it can get without it being in the real world.

### Modes of Intention

By asking ourselves where some thing we are testing fits in the following categories, we can minimize the amount of tests and the sprawl of tests that do seem justified.

- Integrated
    - Intrinsically
    - Practically
    - Marginally
- Secluded
    - Optimally
    - Fractionally
    - Nominally
- Expected
    - Mutually
    - Externally
    - Internally

Initially, it may seem like "Integrated" and "Secluded" are somewhat redundant. An example that shows that they are not would be when a test is intended to be focused on a single unit, but, for pragmatic reasons, that unit happens to access environment variables. We would say that the test is "marginally integrated", yet it is "fractionally secluded". By having and coming to terms with these things, we can more easily focus on the true goals of software testing. 
