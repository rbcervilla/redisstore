# RedisStore

A [`Gorilla Sessions Store`](https://www.gorillatoolkit.org/pkg/sessions#Store) implementation backed by Redis.

It uses [`go-redis`](https://github.com/go-redis/redis) as client to connect to Redis.

If you are using **Redis 6** (client go-redis/v8), install redisstore/v8:

```shell
go get github.com/rbcervilla/redisstore/v8
```

If you are using **Redis 7** (client go-redis/v9), install redisstore/v9:

```shell
go get github.com/rbcervilla/redisstore/v9
```

## Example
```go

package main

import (
    "context"
    "github.com/go-redis/redis/v9"
    "github.com/gorilla/sessions"
    "github.com/rbcervilla/redisstore/v9"
    "log"
    "net/http"
    "net/http/httptest"
)

func main() {

    client := redis.NewClient(&redis.Options{
        Addr:     "localhost:6379",
    })

    // New default RedisStore
    store, err := redisstore.NewRedisStore(context.Background(), client)
    if err != nil {
        log.Fatal("failed to create redis store: ", err)
    }

    // Example changing configuration for sessions
    store.KeyPrefix("session_")
    store.Options(sessions.Options{
        Path:   "/path",
        Domain: "example.com",
        MaxAge: 86400 * 60,
    })

    // Request y writer for testing
    req, _ := http.NewRequest("GET", "http://www.example.com", nil)
    w := httptest.NewRecorder()

    // Get session
    session, err := store.Get(req, "session-key")
    if err != nil {
        log.Fatal("failed getting session: ", err)
    }

    // Add a value
    session.Values["foo"] = "bar"

    // Save session
    if err = sessions.Save(req, w); err != nil {
        log.Fatal("failed saving session: ", err)
    }

    // Delete session (MaxAge <= 0)
    session.Options.MaxAge = -1
    if err = sessions.Save(req, w); err != nil {
        log.Fatal("failed deleting session: ", err)
    }
}

```