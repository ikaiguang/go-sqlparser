# go-sqlparser

sqlparser, SQL Parser, go sql parser, golang sql parser

摘自 [https://github.com/vitessio/vitess](https://github.com/vitessio/vitess)

## 版本

```bash

go version go1.11.1 darwin/amd64

cd $GOPATH/src/github.com/vitessio/vitess

git log

```

```text

commit 32dd398dd5459e5cf15904db35aba7460519b612 (HEAD -> master, origin/master, origin/HEAD)
Merge: 714e12689 066779f52
Author: Michael Demmer <demmer@gmail.com>
Date:   Wed Nov 21 21:19:16 2018 -0500

    Merge pull request #4336 from planetscale/ss-vtgatev3
    
    select *: expand column list for tables with authoritative column lists

commit 714e126899c76a821dac78148f2d51550621edaa
...

```
## Usage

import (
    "github.com/ikaiguang/go-sqlparser"
)

## example

[parse_test.go](parse_test.go)

[parse_next_test.go](parse_next_test.go)
...

## my example

```go

// TestParse Parse
func TestParse(t *testing.T) {
	return

	src := "CREATE TABLE foo (a SMALLINT UNSIGNED, b INT UNSIGNED); -- foo\nSelect --1 from foo;"

	stat, err := sqlparser.Parse(src)
	//stat, err := sqlparser.ParseStrictDDL(src)
	if err != nil {
		t.Errorf("sqlparser.Parse error : %v \n", err)
		return
	}
	t.Logf("sqlparser.Parse success \n")
	t.Logf("%#v \n", stat)

	ddl, ok := stat.(*sqlparser.DDL)
	if !ok {
		t.Errorf("stat.(*sqlparser.DDL) convert fail")
		return
	}
	t.Logf("%#v \n", ddl)
}

// TestTokenizer Tokenizer
func TestTokenizer(t *testing.T) {
	return

	src := `
CREATE TABLE foo (a SMALLINT UNSIGNED, b INT UNSIGNED);
CREATE TABLE bar (a SMALLINT UNSIGNED, b INT UNSIGNED);
CREATE TABLE bar (a SMALLINT UNSIGNED, b INT UNSIGNED);
`
	tokenizer := sqlparser.NewStringTokenizer(src)

	for {
		stat, err := sqlparser.ParseNext(tokenizer)
		if err != nil {
			if err == io.EOF {
				t.Logf("parse done !")
				break
			}
			t.Errorf("sqlparser.ParseNext(tokenizer) fail : %v", err)
			break
		}
		ddl, ok := stat.(*sqlparser.DDL)
		if !ok {
			t.Errorf("stat.(*sqlparser.DDL) convert fail")
			return
		}
		t.Logf("%#v \n", ddl)
	}
}

// TestSplitStatement SplitStatement
func TestSplitStatement(t *testing.T) {
	return

	src := `
CREATE TABLE foo (a SMALLINT UNSIGNED, b INT UNSIGNED);
CREATE TABLE bar (a SMALLINT UNSIGNED, b INT UNSIGNED);
CREATE TABLE bar (a SMALLINT UNSIGNED, b INT UNSIGNED);
`
	for len(src) > 0 {
		// get one
		str1, str2, err := sqlparser.SplitStatement(src)
		if err != nil {
			if err == io.EOF {
				t.Logf("split done !")
				break
			}
			t.Errorf("sqlparser.SplitStatement(src) fail : %v", err)
			break
		}
		t.Logf("%#v \n", str1)
		//t.Logf("%#v \n", str2)

		src = str2
	}
}

// TestSplitStatementToPieces SplitStatementToPieces
func TestSplitStatementToPieces(t *testing.T) {
	//return

	src := `
-- a
CREATE TABLE foo (a SMALLINT UNSIGNED, b INT UNSIGNED);
-- b
CREATE TABLE bar (a SMALLINT UNSIGNED, b INT UNSIGNED);
-- c
#CREATE TABLE ccc (a SMALLINT UNSIGNED, b INT UNSIGNED);
-- d
#CREATE TABLE ccc (a SMALLINT UNSIGNED, b INT UNSIGNED);
`
	pieces, err := sqlparser.SplitStatementToPieces(src)
	if err != nil {
		t.Errorf("sqlparser.SplitStatementToPieces(src) fail : %v", err)
		return
	}
	t.Logf("%#v \n", len(pieces))
	t.Logf("%#v \n", pieces)

	t.Logf("%#v \n", "~~~~~")

	for _, sql := range pieces {
		//stat, err := sqlparser.Parse(sql) // last sql error
		stat, err := sqlparser.ParseStrictDDL(sql) // last sql error
		if err != nil {
			t.Errorf("sqlparser.Parse error : %v \n", err)
			return
		}

		ddl, ok := stat.(*sqlparser.DDL)
		if !ok {
			t.Errorf("stat.(*sqlparser.DDL) convert fail")
			return
		}
		t.Logf("%#v \n", ddl)
	}
}

```
