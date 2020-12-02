# Live Data Broker

## Motivation
In order to deliver live data without having many clients constantly polling from our API endpoints and causing high
system load, it would be great to be able to push updates to all website visitors.

## Concept
The broker works as a middleman between a traditional pull based API and turns it into a websocket service which broadcasts
data to many clients.

Clients may subscribe callbacks to different endpoints. Whenever new data arrives from the broker, the callback is called.
An endpoint is identified with a string. The string may include a dynamic part like an identifier.  

### Clientside Example
```typescript
import liveClient from "live-data-broker/client";

liveClient.subscribe("game/12", (data) => {
    console.log(data);
});
```

Serverside, the `publish` function is being called to create endpoints where clients can subscribe to. The provided callback
will be called when the first client subscribes to the endpoint. The callback sets up periodical data fetching and calls the `update`
method with fresh data. When the data has changed since the last broadcast, an update will be issued. The callback needs to return another
callback that is executed when the last client stopped observing the endpoint and cleans up any intervals or other data fetching.

When a new client subscribes to the endpoint again at a later time, the setup callback will be called again, re-enabling the data fetching.

### Serverside Example
```typescript
const fetch = require("node-fetch");
const broker = require("live-data-broker/server");

broker.publish("game/[id]", ({id}, update) => {
    const interval = setInterval(() => {
        fetch(`https://example.com/api/games/${id}`).then(update);
    }, 5000);
    
    return () => {
        clearInterval(interval);
    }   
});
```
