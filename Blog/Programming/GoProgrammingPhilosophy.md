# Project Style Guide

## Service Structure Philosophy
The project follows functional-programming like structure.
The main idea is to have a clear separation of code based on data operation and stages.

This project treats every service as a data transformation pipeline.
In a micro backend service, every piece of code can be classified into following categories:
1. **Declaration** - Structure definition.
2. **Transformation** - Structure initialization and logic operations.

Therefore, the main service code is seperated into following root folders:
### Declaration
- **Model** - Data model. Responsible for data structure and type definition.
### Transformation
- **Const** - Data constant. Responsible for providing access to constant data.
- **Controller** - Data controller. Responsible for data construction and transformation.

# Rules
## File Placement
Categorized by code functionality.
- **Type Definition** - Model Folder
- **Data Initialization** Controller
- **Function** Controller
## Coding Rules
### ELIMINATE LOGIC & STRUCTURE AMBIGUITY
Logic and structure ambiguity should be hated upon as different coder has different thought process.
Enabling the scenario where two coder writes code conflicting code, but both are them approaches the problem correctly.
When both coders are right and conflict of decision happens, that means the coding rule is bad.
### ALL Parameters Are Immutable
It will create redundant code & lines, but the rules help contribute to fool-proof code.
### Member Function Cannot Take Parameter

Consider the following example:
```go
package example

type ApplePen struct {
}
type Apple struct {
}
type Pen struct {
}

func (a *Apple) Ugh(p Pen) ApplePen {return ApplePen{}}
func (p *Pen) Ugh(a Apple) ApplePen {return ApplePen{}}
```
From data transformation standpoint both pen and apple are information needed to assemble apple pen.
Though in real world there needs to be a "primary data body" for the call to happen on.
In order to solve this, people might introduce a interface called ApplePenBuilder and takes both apple and pen as input.
Which is redundant, ugly and confusing.

To solve this, the project forbid the use of none-zero member function.
Instead, when writing code, start from the data structure and think about the data transformation.
This is what the code will look like without interface:

```go
package example

type ApplePen struct {
    Apple Apple
    Pen   Pen
}
type Pen struct {
}
type Apple struct {
}

func Ugh(apple Apple, pen Pen) ApplePen {return ApplePen{Apple{}, Pen{}}}
```

### No Interface

The most important part of interface is unification of data operation.
Consider the following example:
```go
package example

type Result struct {}
type Client interface {
	Call() Result
}

type httpClient struct {}
type httpsClient struct {
	option1 string
	option2 string
}

func (h *httpClient) Call() Result {return Result{}}
func (h *httpsClient) Call() Result {return Result{}}
```

Use of interface also introduce similar problem as above: obscurity of primary operator.
Consider web accessor `Client` and request target `Url`. There are two ways to structure this:
```go
package example

type Result string
type Url string
type Client interface {
	RequestTo(string Url) Result
}
```

```go
package exapmle

type Result string
type Client struct {}
type Url interface {
	RequestWith(c Client) Result
}
```
No one would write the project the second way, but it is structurally correct.
This means such example will happen in more complex situations.
Meaning that structural ambiguity will occur.

In this project, interface unification is achieved through function pointer:

```go
package example

type Result struct {}
type Client struct {
	call func() Result
}

func CallHttp() Result {return Result{}}
func CallHttps(option1 string, option2 string) Result {return Result{}}

func ProvideHttpClient() Client {
	return Client{CallHttp}
}
func ProvideHttpsClient(option1 string, option2 string) Client {
	// Reorganization/separation is desirable here
	clientCall := func() Result {CallHttps(option1, option2)}
	return Client{clientCall}
}
```
