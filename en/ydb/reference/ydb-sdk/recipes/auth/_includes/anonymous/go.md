---
sourcePath: en/ydb/ydb-docs-core/en/core/reference/ydb-sdk/recipes/auth/_includes/anonymous/go.md
---
```go
package main

import (
	"context"
	
	"github.com/ydb-platform/ydb-go-sdk/v3"
)

func main() {
	ctx, cancel := context.WithCancel(context.Background())
	defer cancel()
	db, err := ydb.New(
		ctx,
		...
		ydb.WithAnonymousCredentials(),
	)
	if err != nil {
		panic(err)
	}
	defer func() { 
		_ = db.Close(ctx) 
	}()
}
```
