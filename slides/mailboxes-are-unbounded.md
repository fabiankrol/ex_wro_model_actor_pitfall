# mailboxes are unbounded 😱

====

### fabian krol
### @fabiankrol

====

### spawn - send
```elixir
iex()> send self(), "msg1"
iex()> send self(), "msg2"
iex()> flush
"msg1"
"msg2"
```

====

### spawn - send - mailbox
```elixir
iex()> send self(), "msg3"
iex()> send self(), "msg4"
iex()> Process.info self
...
iex()> Process.info self, :messages
{:messages, ["msg3", "msg4"]}
iex()> Process.info self, :message_queue_len
{:message_queue_len, 2}
```

====

### spawn - send - receive
```elixir
iex()> pid = spawn fn ->
...()>   receive do
...()>     msg -> IO.inspect "message: #{msg}"
...()>   end
...()> end
iex()> send pid, "msg1"
"message: msg1"
```

====

#### spawn - send - selective receive
```
iex()> pid = spawn fn ->
...()>   receive do
...()>     {:ok, msg} -> IO.inspect "message: #{msg}"
...()>   end
...()> end
iex()> send pid, "test1"
iex()> Process.info pid, :messages
{:messages, ["test1"]}
iex()> send pid, {:ok, "test3"}
"message: test3"
```

====

# mailboxes are unbounded 😱

====

#### problem
```
iex()> pid = spawn fn ->
...()>   receive do
...()>     {:ok, msg} -> IO.inspect "get: #{msg}"
...()>   end
...()> end
iex()> loop = fn next -> send pid, :msg; next.(next) end
iex()> spawn fn -> loop.(loop) end

iex()> Process.info pid, :message_queue_len
{:message_queue_len, 1050578}
iex()> Process.info pid, :message_queue_len
{:message_queue_len, 1355752}
iex()> Process.info pid, :message_queue_len
{:message_queue_len, 1520659}
```

====

## solution

### rate limit on the producer
```
    jlouis/fuse
    awochna/breaker
```

### dropping messages on the receiver
```
    ferd/pobox
```

