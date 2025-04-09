# Current Notes:

## Arranging

Expressing progression.

### Functionalization
- Formation of Values
    - Construction   (constructor [type return], calculator, getter)
    - Articulation   (constructor [func return])
    - Modification   (setter)
- Relation of Behavior
    - Exposition     (handler)
    - Junction       (multiplexer)
    - Seclusion      (subhandler)
- Notation of Attributes
    - Identification (discriminator [no return])
    - Elucidation    (getter [static return])

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

func Name(a *App) string { // no processing
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

### Structuratlization
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

func NewDBDetails(host string, port int, user, pass string) *DBaseDSN {
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
	db     *sql.DB
}

func NewApp(db *sql.DB) *App {
    return &App{
        db:     db,
    }
}

func (a *App) PingDB() error {
    return a.db.Ping()
}
```

## Conditioning

Expressing digression.

### Typification
- Pointer
- Enums
- Bit shifted integers (rwx)
- Special meaning integers (-1)

### Disjunction
- Error (XOR Value)
    - Token (simple, "is")
    - Behavior (complex, "as")

### Resignation
- Panic
- Abrupt return

## Testing

Verifying congression.

### Modes
- Expected
    - Mutually
    - Externally
    - Internally
- Integrated
    - Thoroughly
    - Partially
    - Minimally
- Pinned
    - Tightly
    - Moderately
    - Loosely

### Scopes
- Unit
- Surface
- System
