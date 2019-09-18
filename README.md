# RedisStore

A [`Gorilla Sessions Store`](https://www.gorillatoolkit.org/pkg/sessions#Store) implementation backed by Redis.

It uses [`go-redis`](https://github.com/go-redis/redis) as client to connect to Redis.

## Example
``` go
client := redis.NewClient(&redis.Options{
    Addr:     "localhost:6379",
})

// New default RedisStore
store, err := NewRedisStore(client)
if err != nil {
    t.Fatal("failed to create redis store: ", err)
}

// Get session
session, err = store.Get(req, "session-key")
if err != nil {
	 t.Fatal("failed getting session: ", err)
}

// Add a value
session.Values["foo"] = "bar"

// Save session
if err = sessions.Save(req, w); err != nil {
	 t.Fatal("failed saving session: ", err)
}

// Delete session (MaxAge <= 0)
session.Options.MaxAge = -1
if err = sessions.Save(req, w); err != nil {
	t.Fatal("failed deleting session: ", err)
}

// Change configuration for new sessions
store.KeyPrefix("session_keyprefix_")
store.Options(sessions.Options{
		Path:   "/path",
		Domain: "example.com",
		MaxAge: 86400 * 60,
	})
```