---
tags: [golang, cheatsheet]
---

# Go — Sintaxe Essencial

[[00 - HOME|Home]] · [[Go - Idiomas e Convenções]]

## Variáveis, constantes, iota
```go
var a int = 1
b := "go"            // só dentro de funções
const Pi = 3.14

type Role int
const (
    Admin Role = iota // 0
    User              // 1
    Guest             // 2
)
```

## Funções, múltiplos retornos, defer
```go
func divide(a, b float64) (float64, error) {
    if b == 0 {
        return 0, errors.New("divisão por zero")
    }
    return a / b, nil
}

func read(path string) error {
    f, err := os.Open(path)
    if err != nil {
        return fmt.Errorf("abrir %s: %w", path, err)
    }
    defer f.Close()
    // ...
    return nil
}
```

## Slices e maps
```go
s := []int{1, 2, 3}
s = append(s, 4)
sub := s[1:3]                // partilha o backing array!
cp := slices.Clone(s)        // cópia independente

m := map[string]int{"a": 1}
if v, ok := m["b"]; !ok { _ = v }
delete(m, "a")
```

## Structs, métodos, ponteiros
```go
type Account struct {
    ID      string
    balance int64
}

func NewAccount(id string) *Account { return &Account{ID: id} }

func (a *Account) Deposit(n int64) error {
    if n <= 0 { return ErrInvalidAmount }
    a.balance += n
    return nil
}
func (a Account) Balance() int64 { return a.balance }
```

## Interfaces e type switch
```go
type Notifier interface {
    Notify(ctx context.Context, msg string) error
}

switch v := x.(type) {
case string: fmt.Println("string", v)
case int:    fmt.Println("int", v)
default:     fmt.Println("outro")
}
```

## Erros
```go
var ErrNotFound = errors.New("não encontrado")

type ValidationError struct{ Field string }
func (e ValidationError) Error() string { return "campo inválido: " + e.Field }

if errors.Is(err, ErrNotFound) { /* ... */ }
var ve ValidationError
if errors.As(err, &ve) { /* ... */ }
```

## Generics
```go
func Map[T, U any](in []T, f func(T) U) []U {
    out := make([]U, 0, len(in))
    for _, v := range in { out = append(out, f(v)) }
    return out
}

type Number interface{ ~int | ~int64 | ~float64 }
func Sum[T Number](xs []T) (s T) { for _, x := range xs { s += x }; return }
```

## Concorrência
```go
var wg sync.WaitGroup
results := make(chan int)

for i := 0; i < 3; i++ {
    wg.Add(1)
    go func() {
        defer wg.Done()
        results <- i * i
    }()
}
go func() { wg.Wait(); close(results) }()

for r := range results { fmt.Println(r) }

ctx, cancel := context.WithTimeout(context.Background(), 2*time.Second)
defer cancel()
select {
case <-ctx.Done(): fmt.Println(ctx.Err())
case r := <-results: fmt.Println(r)
}
```

## HTTP mínimo (Go 1.22+)
```go
mux := http.NewServeMux()
mux.HandleFunc("GET /users/{id}", func(w http.ResponseWriter, r *http.Request) {
    id := r.PathValue("id")
    json.NewEncoder(w).Encode(map[string]string{"id": id})
})

srv := &http.Server{
    Addr:              ":8080",
    Handler:           mux,
    ReadHeaderTimeout: 5 * time.Second,
}
log.Fatal(srv.ListenAndServe())
```

## Teste table-driven
```go
func TestDivide(t *testing.T) {
    tests := []struct {
        name    string
        a, b    float64
        want    float64
        wantErr bool
    }{
        {"normal", 6, 3, 2, false},
        {"por zero", 1, 0, 0, true},
    }
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            got, err := divide(tt.a, tt.b)
            if (err != nil) != tt.wantErr { t.Fatalf("err=%v", err) }
            if got != tt.want { t.Errorf("got %v want %v", got, tt.want) }
        })
    }
}
```
