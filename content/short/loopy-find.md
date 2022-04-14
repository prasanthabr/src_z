+++
title = "Loopy Find"
date = 2022-04-15T11:23:53+12:00
description = "simple script build to find a keyset based on results which we want to target"
draft = false
toc = true
categories = ["short"]
tags = ["go", "csv", "recursive"]
images = [
"https://source.unsplash.com/collection/983219/1600x900"
] # overrides site-wide open graph image
[[copyright]]
owner = "prasanthabr"
date = "2022"
license = "cc-by-sa-4.0"
+++

{{% external rel="next" href="https://source.unsplash.com/collection/983219/2160x1440" %}}
![Example image](https://source.unsplash.com/collection/983219/1080x720 "View Random Image Enlarged")
{{% /external %}}

{{< hackcss-card header="background" >}}This is a custom scripts to <mark>recursively filter records similar to an index</mark> on a CSV.
Though the script, then filters down to a series of results that do have the matching result.{{< /hackcss-card >}}

# Mechanism

This is pretty basic, with a couple of functions doing the heavy lift.

First the code, creates a map of the find key and recursive results that match that key. This is admittedly inefficient, but been done for cases where the key does not exist as the first available value. _will think about a better algo to make this more efficient._

The matching key is being bled into an array, which is then used to filter out the map.

The final results are then set into a 2x2 array and printed into a CSV.

# Learnings

How do I transpose the CSV?
How do we split out the reading of the file, without entering an infinite loop _this should be easy, and just needs some thought_

```go
package main

import (
	"encoding/csv"
	"fmt"
	"io"
	"log"
	"os"
	"sort"
)

func main() {
	programmap("ace.csv", "AK1305", 1, 0)

}

func programmap(file string, prog string, course int, student int) {
	var header []string
	cxmap := make(map[string][]string) //map of student
	var stds []string
	cxfinal := [][]string{}
	count := 0

	csvFile, err := os.Open(file)
	if err != nil {
		log.Fatal(err)
	}

	f := csv.NewReader(csvFile)

	for {

		if count > 50000 {
			break
		}
		record, err := f.Read()

		if err == io.EOF {
			break
		}

		count++

		if err != nil {
			log.Fatal(err)
		}

		if header == nil {
			header = record
			continue
		}

		cxmap[record[student]] = append(cxmap[record[student]], record[course])

		if record[course] == prog {
			stds = append(stds, record[student])
			fmt.Println(cxmap[record[student]])

		}
	}

	sort.Strings(stds)

	for _, v := range stds {
		temp := make([]string, 0)
		temp = append(temp, v)
		sort.Strings(cxmap[v])
		temp = append(temp, cxmap[v]...)
		cxfinal = append(cxfinal, [][]string{temp}...)

	}

	writecsv(cxfinal)
}

func writecsv(rows [][]string) {

	csvfile, err := os.Create("test.csv")

	if err != nil {
		log.Fatalf("failed creating file: %s", err)
	}

	csvwriter := csv.NewWriter(csvfile)

	for _, row := range rows {
		_ = csvwriter.Write(row)
	}

	csvwriter.Flush()

	csvfile.Close()

}



```
