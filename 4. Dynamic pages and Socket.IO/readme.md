# 4. Dynamic pages and Socket.IO
This section shows how to create dynamic page and communicate between frontend and backend via [http://Socket.io/](http://socket.io/).

It requires _Initial setup_ to be done and the sources from "3. Browserify modules" folder.

## Dynamic page
So far our HTTP service hosted only static documents, like `index.html`, etc. Now we will add dynamic page.

Add following code to `service.js`:
```
// === Dynamic page ===
// Load our package definition as an object
var packageInfo = require('./package.json');

// Generate dynamic page on GET request
app.get('/about.html', function(req, res) {
  var currentTime = new Date().toLocaleTimeString();
  res.send("<strong>" + packageInfo.name + "</strong><br />version: " + packageInfo.version + "<br />License: " + packageInfo.license + "<br />Backend local time: " + currentTime);
});
```
Start service and open [http://localhost:5000/about.html](http://localhost:5000/about.html) to see generated message. 
Refresh it few times to see that message is dynamically generated each time you're requesting the address.
You can also put a breakpoint to `var currentTime = new Date().toLocaleTimeString();` line to catch request on backend side.

## Communication between the frontend and the backend
Socket.IO has two flavors `socket.io` for Node.JS side and `socket.io-client` for the the browser side.
Install them both:
```
npm install --save socket.io
npm install --save socket.io-client
```

Add an input text box, a button and a div for output to the `static/index.html` after `Hello world!` line:
```
<div>
  <input id="input-user" value="write your message here"></input>
  <button id="button-send">Send to server</button><br />
  <div id="div-server-message">Server's response will appear here</div>
</div>
```

Add handlers to the frontend's code in `index.js`:
```
// === Communication with server over Socket.IO
// Create connection
var io = require('socket.io-client').connect();

$( document ).ready(function() {
    // Send whatever we have in text box on button click
    $('#button-send').click(function() {
        var userText = $('#input-user').val();
        io.emit('user message', userText);
    });

    // Update output div on server message
    io.on('server message', function (data) {
        $('#div-server-message').html(data);
    });
});
```
Don't forget to bundle client code via `npm run-script build` command.

Add backend code to the `service.js`:
```
// === Communication with frontend via Socket.IO ===
var io = require('socket.io')(server);

// When client connects
io.on('connection', function(client){
  console.log("client connected: " + client.conn.remoteAddress);
  // Add handler for that client
  client.on('user message', function (data) {
    client.emit('server message', "Server received following data: <em>" + data + "</em>")
  });
});
```
It uses `server` instance of HTTP server, we're already using for `express`.

Start backend now (_View -> Debug_, choose `Launch` from combobox and press `F5`) and open [http://localhost:5000/](http://localhost:5000/).