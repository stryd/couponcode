# CouponCode for Go

An implementation of Perl's [Algorithm::CouponCode][couponcode] for Golang.

# Synopsis #

A 'CouponCode' is a type of code that will be passed *in printed form* to
someone who will be expected to type it into a web page or other application.
The codes are random strings which are designed to be easy for the recipient to
type accurately into a web form.

The following features make the codes well suited to manual transcription:

* The codes are not case sensitive.
* Not all letters and numbers are used, so if a person enters the letter 'O' we
  can automatically correct it to the digit '0' (similarly for I => 1, S => 5,
  Z => 2).
* The 4th character of each part is a checkdigit, so client-side scripting can
  be used to highlight parts which have been mis-typed, before the code is even
  submitted to the application's back-end validation.
* The checkdigit algorithm takes into account the position of the part being
  keyed. So for example '1K7Q' might be valid in the first part but not in the
  second so if a user typed the parts in the wrong boxes then their error could
  be highlighted.
* The code generation algorithm avoids 'undesirable' codes. For example any
  code in which transposed characters happen to result in a valid checkdigit
  will be skipped. Any generated part which happens to spell an 'inappropriate'
  4-letter word (e.g.: 'P00P') will also be skipped.

# Usage #

The package provides a default configuration for codes of 2 parts of 4 characters separated by a '-'. The default
configuration can be accessed using the `Default` package field or, for convenience, two top level functions:

`Generate() string` to generate a coupon code
`Validate(code string) (string, error)` to normalize and validate a code

```
package couponcode_test

import (
	"fmt"

	"github.com/captaincodeman/couponcode"
)

func main() {
	code := couponcode.Generate()
	fmt.Println(code)
	// Output: RCMD-CRVF
}
```

To use a custom part and part length, create a new generator using the constructor and then use the functions
provided by that instance:

```
cc := couponcode.New(4, 6)

code := cc.Generate() // note function of cc, not couponcode package

validated, err := cc.Validate(code)
```

Now, when someone types their code in, you can check that it is valid. This means that letters like `O`
are converted to `0` prior to checking. The validate command returns a normalized code (useful for any
consistent database lookups of the coupon code) and an error to indicate if the code was valid.

```
// same code, just lowercased
code, err := couponcode.Validate('55g2-dhm0-50nn');
// '55G2-DHM0-50NN'

// various letters instead of numbers
code, err := couponcode.Validate('SSGZ-DHMO-SONN');
// '55G2-DHM0-50NN'

// wrong last character
code, err := couponcode.Validate('55G2-DHM0-50NK');
// err != nil

// not enough chars in the 2nd part
code, err := couponcode.Validate('55G2-DHM-50NN');
// err != nil
```

The first thing we do to each code is uppercase it. Then we convert the following letters to numbers:

* O -> 0
* I -> 1
* Z -> 2
* S -> 5

This means [oizs], [OIZS] and [0125] are considered the same code.

# Example #

Let's say you want a user to verify they got something, whether that is an email, letter, fax or carrier pigeon. To
prove they received it, they have to type the code you sent them into a certain page on your website. You create a code
which they have to type in:

```
code := couponcode.Generate();
// 55G2-DHM0-50NN
```

Time passes, letters get wet, carrier pigeons go on adventures and faxes are just as bad as they ever were. Now the
user has to type their code into your website. The problem is, they can hardly read what the code was. Luckily we're
somewhat forgiving since Z's and 2's are considered the same, O's and 0's, I's and 1's and S's and 5's are also mapped
to each other. But even more than that, the 4th character of each group is a checkdigit which can determine if the
other three in that group are correct. The user types this:

```
[s5g2-dhmo-50nn]
```

Because our codes are case insensitive and have good conversions for similar chars, the code is accepted as correct.

Also, since we have a checkdigit, we can use a client-side plugin to highlight to the user any mistake in their code
before they submit it. Please see the original project ([Algorithm::CouponCode][couponcode]) for more details of client
side validation.

# Installation

The easiest way to get it is via `go get`:

``` bash
$ go get -u github.com/stryd/couponcode
```

# Tests

To run the tests, use go test:

```
$ go test
```
