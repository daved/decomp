# Decomp: Modular, Maintainable, Testable Code

The most common primitive used to express behavior is the function, yet we do not have nomenclature for decomposing functions when they grow unwieldy, whether that be technically or semantically. It may seem unimportant to dig into the whys of functions and data structures. However, we name complex patterns like Singleton and Bridge. We also formalize paradigms like Functional Programming with rigorous math. This easily missed void is one we can, and should, address. This document is intended to kick-off a common language for code decomposition with the hope of realizing a sharper engineering mindset in software development.

Decomposing code involves breaking down behavior and data into the most relevant and purposeful elements. Composing code involves integrating those well-decomposed elements minimally and effectively. The following "categories of purpose" provide practical, adaptable guidelines through clear, structured naming. They cover arranging behavior and data, conditioning desired control flows with error handling, and testing source code.

## Arranging

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

#### Formation of Values (Examples)

##### Construction

###### Constructor

```go
type App struct {
	Name string
}

func NewApp(name string) *App {
	return &App{Name: name}
}
```

###### Calculator

```go
func add(a, b int) int {
	return a+b
}
```

###### Getter

```go
type App struct {
	Name string
}

func Name(a *App) string { // without processing
	return a.Name
}

func Greeting(a *App) string { // with processing
	name := a.Name
	if name == "" {
		name = "App"
	}
	return "Hello from " + name
}
```

##### Articulation

###### Constructor

```go
func Accumulation(initVal int) func(addend int) int {
	return func(a int) int {
		initVal += a
		return initVal
	}
}
```

##### Modification

###### Setter

```go
type App struct {
	name string
}

func SetName(a App, name string) App {
	a.name = name
	return a
}
```

#### Relation of Behavior (Examples)

##### Exposition

###### Handler

```go
func HandleHello(w http.ResponseWriter, r *http.Request) {
	if !isAuthorized(r) {
		http.Error(w, "Unauthorized", http.StatusUnauthorized)
		return
	}
	name := nameFromRequest(r)
	fmt.Fprintf(w, "Hello, %s!", name)
}
```

##### Junction

###### Multiplexer

```go
func RouteHandling(data []byte) error {
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

###### Subhandler

```go
func HandleHello(w http.ResponseWriter, r *http.Request) {
	// other behavior ...
	PrintHello(w, r)
}

func PrintHello(w http.ResponseWriter, r *http.Request) {
	name := nameFromRequest(r)
	fmt.Fprintf(w, "Hello, %s!", name)
}
```

#### Notation of Attributes (Examples)

##### Identification

###### Discriminator

```go
type ExitError struct {
	Code int
}

// other methods...

func (e *ExitError) Silent() {} // presence conveys type attribute
```

##### Elucidation

###### Getter

```go
type ExitError struct {
	Code int
}

// other methods...

func (e *ExitError) IsSilent() { // value conveys type attribute
	return false
}
```

### Structuralization

- Formalization of Data
    - Inscription       (direct model)
    - Stylization       (abstract model)
- Association of Types
    - Contextualization (dependencies)

#### Formalization of Data (Examples)

##### Inscription

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

#### Association of Types (Examples)

##### Contextualization

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

- Pointer
- Enums
- Bit-shifted Integers (rwx)
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

- Error [XOR Value]
    - Token [simple, "is"]
    - Behavior [complex, "as"]

#### Error [XOR Value] (Examples)

##### Token [simple, "is"]

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

##### Behavior [complex, "as"]

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

Here we cover how tests can be bound by expectations, and relationships existing in the targeted functions and types.

### Scopes

- Unit    (function)
- Surface (API - object, package/module, etc.)
- System  (system/artifact)

### Modes

- Expected
    - Mutually
    - Externally
    - Internally
- Integrated
    - Thoroughly
    - Partially
    - Minimally
