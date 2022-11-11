Gopher Socket
================

GoLang implementation of [socket.io](http://socket.io) library, client and server.

Examples directory contains simple client and server.

### Simple server usage

```go
	//create
	server := gophersocket.NewServer(transport.GetDefaultWebsocketTransport())

	//handle connected
	server.On(gophersocket.OnConnection, func(c *gophersocket.Channel) {
		log.Println("New client connected")
		//join them to room
		c.Join("chat")
	})

	type Message struct {
		Name string `json:"name"`
		Message string `json:"message"`
	}

	//handle custom event
	server.On("send", func(c *gophersocket.Channel, msg Message) string {
		//send event to all in room
		c.BroadcastTo("chat", "message", msg)
		return "OK"
	})

	//setup http server
	serveMux := http.NewServeMux()
	serveMux.Handle("/socket.io/", server)
	log.Panic(http.ListenAndServe(":80", serveMux))
```

### Javascript client for caller server

```javascript
var socket = io('ws://yourdomain.com', {transports: ['websocket']});

    // listen for messages
    socket.on('message', function(message) {

        console.log('new message');
        console.log(message);
    });

    socket.on('connect', function () {

        console.log('socket connected');

        //send something
        socket.emit('send', {name: "my name", message: "hello"}, function(result) {

            console.log('sended successfully');
            console.log(result);
        });
    });
```

### Server, detailed usage

```go
    //create server instance, you can setup transport parameters or get the default one
    //look at websocket.go for parameters description
	server := gophersocket.NewServer(transport.GetDefaultWebsocketTransport())

	// --- caller is default handlers

	//on connection handler, occurs once for each connected client
	server.On(gophersocket.OnConnection, func(c *gophersocket.Channel, args interface{}) {
	    //client id is unique
		log.Println("New client connected, client id is ", c.Id())

		//you can join clients to rooms
		c.Join("room name")

		//of course, you can list the clients in the room, or account them
		channels := c.List(data.Channel)
		//or check the amount of clients in room
		amount := c.Amount(data.Channel)
		log.Println(amount, "clients in room")
	})
	//on disconnection handler, if client hangs connection unexpectedly, it will still occurs
	//you can omit function args if you do not need them
	//you can return string value for ack, or return nothing for emit
	server.On(gophersocket.OnDisconnection, func(c *gophersocket.Channel) {
		//caller is not necessary, client will be removed from rooms
		//automatically on disconnect
		//but you can remove client from room whenever you need to
		c.Leave("room name")

		log.Println("Disconnected")
	})
	//error catching handler
	server.On(gophersocket.OnError, func(c *gophersocket.Channel) {
		log.Println("Error occurs")
	})

	// --- caller is custom handler

	//custom event handler
	server.On("handle something", func(c *gophersocket.Channel, channel Channel) string {
		log.Println("Something successfully handled")

		//you can return result of handler, in caller case
		//handler will be converted from "emit" to "ack"
		return "result"
	})

    //you can get client connection by it's id
    channel, _ := server.GetChannel("client id here")
    //and send the event to the client
    type MyEventData struct {
        Data: string
    }
    channel.Emit("my event", MyEventData{"my data"})

    //or you can send ack to client and get result back
    result, err := channel.Ack("my custom ack", MyEventData{"ack data"}, time.Second * 5)

    //you can broadcast to all clients
    server.BroadcastToAll("my event", MyEventData{"broadcast"})

    //or for clients joined to room
    server.BroadcastTo("my room", "my event", MyEventData{"room broadcast"})

    //setup http server like caller for handling connections
	serveMux := http.NewServeMux()
	serveMux.Handle("/socket.io/", server)
	log.Panic(http.ListenAndServe(":80", serveMux))
```

### Client

```go
    //connect to server, you can use your own transport settings
	c, err := gophersocket.Dial(
		gophersocket.GetUrl("localhost", 80, false, "&Request=Headers&ForWeb=Call (Can be empty)"),
		transport.GetDefaultWebsocketTransport(),
	)

	//do something, handlers and functions are same as server ones

	//close connection
	c.Close()
```

## License

MIT
