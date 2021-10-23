---
title: 'Golang: Generate Random Numbers'
description: "Here's how to generate pseudorandom numbers in Go between to values. NOTE: You should always seed your random generator, or else it will produce the same result every time."
date: 2021-08-13T11:18:04+02:00
tags: ['go', 'golang']
draft: false
---

Here's how to generate pseudorandom numbers in Go between to values. NOTE: You should always seed your random generator, or else it will produce the same result every time. Include this snippet at the top of your main func.

```golang
func main() {
    rand.Seed(time.Now().UnixNano())

    // ...
}
```

## Generate a random int between two values

This example returns a random integer between min and max, where min is inclusive and max is exclusive.

```golang
func getRandomInt(min, max int) int {
	return rand.Intn(max-min) + min
}

// example
for i := 0; i < 5; i += 1 {
    fmt.Printf("%d ", getRandomInt(5, 10))
}

// output: 8 9 7 6 9
```

## Generate a random int between two values (inclusive)

This example returns a random integer between min and max, where both min and max is inclusive.

```golang
func getRandomIntInclusive(min, max int) int {
	return rand.Intn((max - min + 1)) + min
}

// example
for i := 1; i <= 5; i += 1 {
    fmt.Printf("%d ", getRandomIntInclusive(5, 10))
}

// output: 10 5 6 9 9
```
