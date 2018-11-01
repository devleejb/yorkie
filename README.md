# rotty

## Concept layout

```
 +--Client "A"----------------+
 | +--Document "D-1"--------+ |                +--Server------------------------+
 | | { a: 1, b: [], c: {} } | <---- CRDT ----> | +--Collection "C-1"----------+ |
 | +------------------------+ |                | | +--Document "D-1"--------+ | |
 +----------------------------+                | | | { a: 1, b: [], c: {} } | | |
                                               | | +------------------------+ | | <---> +--Mongo--------+
 +--Client "B"----------------+                | | +--Document "D-2"--------+ | |       | Snapshot      |
 | +--Document "D-1"--------+ |                | | | { a: 1, b: [], c: {} } | | |       | Meta(Ops,...) |
 | | { a: 2, b: [], c: {} } | <---- CRDT ----> | | +------------------------+ | |       +---------------+
 | +------------------------+ |                | +----------------------------+ |
 +----------------------------+                +--------------------------------+
                                                              ^
 +--Client "C" ---------------+                               |
 | +--Query "Q-1"-----------+ |                               |
 | | db.['c-1'].find(...)   | <---- Snapshot Query -----------+
 | +------------------------+ |
 +----------------------------+
```

## Examples

backend

```javascript
const http = require('http');
const WebSocket = require('ws');
const rotty = require('rotty');
const MongoClient = require('mongodb').MongoClient;

// 01. Create a web server to serve files and listen to WebSocket connections
const app = express();
app.use(express.static('static'));
const server = http.createServer(app);

// 02. create rotty backend
const backend = rotty.createFromMongo({
  client: new MongoClient('mongodb://localhost:27017'),
  dbName: 'myproject'
});
  
// 03. Connect any incoming WebSocket connection to rotty
const wss = new WebSocket.Server({server: server});
wss.on('connection', function(ws, req) {
  backend.listen(ws);
});

server.listen(8080);
console.log('Listening on http://localhost:8080');
```

frontend

```javascript
// 01. create a client and connect to the server with it.
const client = rotty.createClient({
  socket: new WebSocket('ws://localhost:8080')
});
await client.connect();

// 02. find a document and change it
const doc1 = await client.collection('documents').attachOne({_id: 'D-1'});
doc1.change(doc => {
  doc.set('a', 2);
});

// 03. close
client.close();
```
