---
title: Writing a CSV With Go
layout: post
categories: [today-i-learned]
---

Today I worked on a task that required me to transform some data into a CSV file that would be attached to an email. Even though I had never generated a CSV with Go before, I estimated that the task wouldn’t take too long — the standard library had to have something to make this easy. Lucky for me, I was right! 

The [csv package](https://golang.org/pkg/encoding/csv/) made this task pretty straightforward. All you need to do is create a `csv.Writer` by passing an  `io.Writer`   destination to `csv.NewWriter()` and then use the `Write()` method to write each record in the CSV.

Here is a fictional example for generating a CSV string of the people who received the Top 10 votes for the Ballon d’Or 2019. For ease of demonstration, here is a [Go Playground link](https://play.golang.org/p/dQgWuh79zTv).
```go
package main

import (
	"bytes"
	"encoding/csv"
	"fmt"
	"log"
)

func main() {
	var buf bytes.Buffer
	w := csv.NewWriter(&buf)

	records := [][]string{
		{"Rank", "Name", "Votes"},
		{"1", "Lionel Messi", "686"},
		{"2", "Virgil van Dijk", "679"},
		{"3", "Cristiano Ronaldo", "476"},
		{"4", "Sadio Mane", "347"},
		{"5", "Mohamed Salah", "178"},
		{"6", "Kylian Mbappe", "89"},
		{"7", "Alisson", "67"},
		{"8", "Robert Lewandowski", "44"},
		{"9", "Bernardo Silva", "41"},
		{"10", "Riyad Mahrez", "33"},
	}

	for _, r := range records {
		if err := w.Write(r); err != nil {
			log.Fatalln("error writing record to csv:", err)

		}
	}

	// csv.Writer performs buffered writes so you need to make sure you flush any unwritten data
	w.Flush()

	// an error might have occured from the flush, check that here
	if err := w.Error(); err != nil {
		log.Fatalln("error flushing writer:", err)
	}

	fmt.Println(buf.String())
}

```
If you wanted to write these results to a file instead, all you would need to do is change the `io.Writer` used to instantiate the `csv.Writer`.
