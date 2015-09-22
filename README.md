# blab
--
    import "github.com/cmcoffee/go-blab"

Package blab provides a means of creating a network of methods/functions.

Callers include producers and consumers, producers may also connect to other
producers. This first Caller to "Listen" doubles as a broker to direct all
Calls. Callers create their own socket file for direct connections to other
Callers.

Once a Call is placed the first Caller/Listener will route the call to the
appropriate producer who registered the method or function. After the reply is
sent, the producer will attempt to create a direct connection to the consumer
for direct answering of any subsiquent calls.

blab has similar requirements for exporting functions that Go's native "rpc"
package provides, however blab maps both functions and object methods.

    1)the method or function requires two arguments, both exported (or builtin) types.
    2)the method or function's second argument is a pointer.
    3)the method or function has a return type of error.

Registered methods or functions should look like:

func (*T) Name(argType T1, replyType *T2) error

    and respectively ...

func name(argType T1, replyType *T2) error

Exported functions & methods should be made thread safe.

## Usage

```go
var ErrClosed = errors.New("Connection closed.")
```

```go
var ErrFail = errors.New("Call to unknown method or function.")
```

#### type Caller

```go
type Caller struct {
   // contains filtered or unexported fields
}
```

The Caller object is for both producers(registering methods/functions) and
consumers(calls registered methods/functions).

#### func  NewCaller

```go
func NewCaller() *Caller
```
Allocates new Caller.

#### func (*Caller) Call

```go
func (cl *Caller) Call(name string, arg interface{}, reply interface{}) (err error)
```
Call invokes a registered method/function, blocks while actively checking for
for completion and returns an error, if any.

#### func (*Caller) Dial

```go
func (cl *Caller) Dial(socketf string) error
```
Creates socket file(socketf) connection to Broker, forks goroutine and returns.
(consumers)

#### func (*Caller) Listen

```go
func (cl *Caller) Listen(socketf string) (err error)
```
Listens to socket files(socketf) for Callers. If socketf is not open, Listen
opens the file and connects itself to it. (producers)

#### func (*Caller) Register

```go
func (cl *Caller) Register(fptr interface{}) error
```
Registers local methods or function, informs Broker of registration.
Function/method template should follow: func name(argType T1, replyType *T2)
error func (*T) Name(argType T1, replyType *T2) error

#### func (*Caller) RegisterName

```go
func (cl *Caller) RegisterName(name string, fptr interface{}) (err error)
```
RegisterName operates exactly as Register but allows changing the name of the
object or function.

#### func (*Caller) SetOutput

```go
func (cl *Caller) SetOutput(w io.Writer)
```
Directs error output to a specified io.Writer, defaults to os.Stdout.