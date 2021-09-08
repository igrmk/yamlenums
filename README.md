# yamlenums

[![Build Status](https://api.travis-ci.com/igrmk/yamlenums.svg)](https://app.travis-ci.com/github/igrmk/yamlenums)
[![GoReportCard](https://goreportcard.com/badge/igrmk/yamlenums)](https://goreportcard.com/report/igrmk/yamlenums)

yamlenums is a tool to automate the creation of methods that satisfy the
`yaml.Marshaler` and `yaml.Unmarshaler` interfaces.
Given the name of a (signed or unsigned) integer type T that has constants
defined, yamlenums will create a new self-contained Go source file implementing

```
func (t T) MarshalYAML() ([]byte, error)
func (t *T) UnmarshalYAML(unmarshal func(v interface{}) error) error
```

The file is created in the same package and directory as the package that
defines T. It has helpful defaults designed for use with go generate.

yamlenums is a copy and a small rework of https://github.com/campoy/jsonenums.

jsonenums is a simple implementation of a concept and the code might not be the
most performant or beautiful to read.

For example, given this snippet,

```Go
package painkiller

type Pill int

const (
	Placebo Pill = iota
	Aspirin
	Ibuprofen
	Paracetamol
	Acetaminophen = Paracetamol
)
```

running this command

```
yamlenums -type=Pill
```

in the same directory will create the file `pill_yamlenums.go`, in package
`painkiller`, containing a definition of

```
func (r Pill) MarshalYAML() ([]byte, error)
func (r *Pill) UnmarshalYAML(unmarshal func(v interface{}) error) error
```

`MarshalYAML` will translate the value of a `Pill` constant to the `[]byte`
representation of the respective constant name, so that the call
`yaml.Marshal(painkiller.Aspirin)` will return the bytes `[]byte("\"Aspirin\"")`.

`UnmarshalYAML` performs the opposite operation;
it unmarshals underlying bytes to a string, given the string
representation of a `Pill` constant it will change the receiver to equal the
corresponding constant. So given the string `"Aspirin"` the receiver will
change to `Aspirin` and the returned error will be `nil`.

Typically this process would be run using go generate, like this:

```
//go:generate yamlenums -type=Pill
```

If multiple constants have the same value, the lexically first matching name
will be used (in the example, Acetaminophen will print as "Paracetamol").

With no arguments, it processes the package in the current directory. Otherwise,
the arguments must name a single directory holding a Go package or a set of Go
source files that represent a single Go package.

The `-type` flag accepts a comma-separated list of types so a single run can
generate methods for multiple types. The default output file is t_yamlenums.go,
where t is the lower-cased name of the first type listed. The suffix can be
overridden with the `-suffix` flag and a prefix may be added with the `-prefix` 
flag.
