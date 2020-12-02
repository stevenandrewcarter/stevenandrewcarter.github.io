---
layout: 	  post
title:  	  Refactoring Unit Tests using Table Driven Tests
date:   	  2020-11-24 18:00:00 +0200
categories: General
published:  true
---

Unit testing is an underused and often forgotten technique for building and debugging software projects. It can be 
difficult to ensure that tests provide value for the code base. Ideally unit tests should ensure that expectations are 
validated and help prevent future changes from breaking the existing code base.

Unit tests can end up as the worst parts of the code base, since its not common to go and refactor the unit tests. 
I feel the reason behind this is that unit tests are the metric to determine if the code base is valid, and changing the
unit tests might introduce breaking changes to the code base. In complex code bases or solutions, updating the code base
might require knowledge about the domain, which might not be easy to gain. Because unit tests end up not getting updated
and maintained, the tests themselves often end up losing value.

The following is a simple application that will show how to take a set of unit tests and refactor the tests to use table
driven approach instead.

```go
package main

import "errors"

func Sum(square bool, a ...int) (int, error) {
	if len(a) == 0 {
		return 0, errors.New("expect at least one item")
	}
	c := 0
	for _, x := range a {
		if square {
			c = c + (x * x)
		} else {
			c = c + x
		}		
	}
	return c, nil
}
```

Trying to make unit tests would end up with the following example. Each test case focuses on a different check based on
the possible inputs, and even with the simple example provided we already have four possible test cases. Each of the 
cases is pretty much the same code.

```go
package main_test

import (
	"testing"

	mathlib "github.com/stevenandrewcarter/table_test"
)

func TestSumReturnsError(t *testing.T) {
	if _, err := mathlib.Sum(false); err == nil {
		t.Fail()
	}
}

func TestSumReturnsError(t *testing.T) {
	if _, err := mathlib.Sum(true); err == nil {
		t.Fail()
	}
}

func TestSumReturnsCorrectValuesWithoutSquare(t *testing.T) {
	if res, err := mathlib.Sum(false, []int{1, 2, 3}...); err != nil {
		t.Errorf("%v", err)
	} else if res != 6 {
		t.Errorf("Expected value %q, Got %q", 6, res)
	}
}

func TestSumReturnsCorrectValuesWithSquare(t *testing.T) {
	if res, err := mathlib.Sum(true, []int{1, 2, 3}...); err != nil {
		t.Errorf("%v", err)
	} else if res != 15 {
		t.Errorf("Expected value %q, Got %q", 15, res)
	}
}
```

The Table driven approach looks as follows.

```go
package main_test

import (
	"testing"

	mathlib "github.com/stevenandrewcarter/table_test"
)

func TestSum(t *testing.T) {
	tests := map[string]struct {
		inputs         []int
		square				 bool
		expectedResult int
		expectedErr    string
	}{
		"requireInputs":        {expectedErr: "expect at least one item"},
		"successfulSquared":    {inputs: []int{1, 2, 3}, square: false, expectedResult: 6},
		"successfulNotSquared": {inputs: []int{1, 2, 3}, square: true, expectedResult: 15},
	}
	for name, tc := range tests {
		t.Run(name, func(t *testing.T) {
			if res, err := mathlib.Sum(tc.square, tc.inputs...); err != nil {
				if tc.expectedErr != err.Error() {
					t.Errorf("Expected error %q, Got %q", tc.expectedErr, err)
				}
			} else if res != tc.expectedResult {
				t.Errorf("Expected value %q, Got %q", tc.expectedResult, res)
			}
		})
	}
}
```
