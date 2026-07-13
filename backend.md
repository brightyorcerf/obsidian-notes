why we need backend: data
reverse proxy, nginx, certbot, cors (explain how these terms relate to backend)
from aws instance pov node server running in local server and we are using nginx domain names to route requests to our local server over internet ??? what does this mean

```
req starts in browser -> dns -> aws firewall -> instance -> nginx -> localhost
- frontend: ??? 
```

why not do every action in frontend instead of using backend?
- browsers have restrictive security policies
- CORS (can't use external APIs)
- databases
- computing power

browser in runtime in the former case, whereas the processing happens in the server in the latter
browser runtimes are sandboxed environments

---
#### Understanding http

- stateless
- client-server model 

self-contained req: since it is stateless each req must have all necessary data (auth token / session info) to handle that specific interaction
benefits: simple + scalable (server doesn't store session info + distribute across several servers)

state management techniques like cookies or sessions to maintain continuity

(client is browser / application that initiates communication by sending req to server and it provides all required info for server; server hosts resources and waits for incoming requests from clients to process and respond)

Historically, http relied on TCP for data transmission; however the modern http/3 uses QUIC, a transport protocol build on top of UDP

>[!tldr]
>Client and Servers establish some kind of are network connection and request / response messages are exchanged. 


http headers are metadata fields sent along to provide context about communication
http methods exist to represent the diff kind of actions that can be req on server
example - get, post, put, patch, delete, options

idempotent: describes property of certain operations where applying them multiple times yields same result as applying them just once
idempotent: get, put, delete; non-idempotent: post, patch

options method is used to fetch the capabilities of server for a cross origin req (CORS) which is used because browsers have same origin policy (if our domain and server domain are diff???)

in cors you have primarily two types of flows:
- simple request // explain diff in one line each
- preflighted request

when is preflighted request fired: 
- method is not get, post or head OR
- includes non-simple headers like auth, x-customer-header OR
- has content-type other than application/x-www-form-urlencoded, multipart/form-data, text/plain 

http response codes exist to communicate result of a req in a standardized way
1xx information, 2xx success, 3xx redirection, 4xx client error, 5xx server error

http caching: storing copies of responses for reuse; (etag, last-modified header), http 304 status
this improves load time, reduces bandwidth, decreases server load

content negotiation: mechanism using which client and sever agree on the best format to exchange data (media, language, encoding and so on)
http compression

persistent connections and keep-alive: 
in early days of http each req-resp cycle required a separate connection now this created inefficiency as establishing and closing tcp connections is resource intensive and slow, to address this persistent connections were introduced; now with this a single tcp connection can be reused for multiple req-resp and for achieving that keep-alive header is used

multipart data and chunked transfer

ssl vs https vs tls
- original encryption between clients and server, currently outdated due to security vulnerabilities
- modern and more secure version of ssl
- https is http + more security which are provided either by tls

---
#### Routing

http methods -> intent / action / what
routing -> where

routing is mapping url parameters to a server-side logic
rest api: sending semantic expressions to api endpoints

// add one line definition + example + use case
static routing
dynamic routing with path parameters
query parameters
nested route
route versioning and deprecation
catch-all route

serialization and deserialization: converting data to a common format during transmission or storage so that it is language / domain agnostic
common standard ex) json in textbased serialization and protobuff for binary format
osi model, javascript client vs rust server


--- 

Authentication & Authorization

who are you? what can you do?

multi-factor auth
- something you know
- something you are 
- something you have

in 21st century where you have cloud computing, mobile devices, api based architectures demanded advanced auth frameworks: oauth 2.0, jwt, zero trust architecture, passwordless

future of auth: decentralized / blockchain, post quantum cryptography 

3 key components
- session: 
- jwt:
- cookie:



--- 
#### RPCs vs edge functions

When building an app using a backend-as-a-service like Supabase, need to choose between RPCs (remote procedure calls) and Edge Functions to handle your backend logic.

- RPCs = Custom SQL function written directly inside your PostgreSQL database. App calls this function via an API client, executing the database code seamlessly.

	sends a request $\rightarrow$ API gateway routes it directly to the database $\rightarrow$ the database executes the SQL logic and returns the data.

	Pros - blazing fast performance, ACID transactions, strong security (RLS)
	Cons - sql only, no external api interactions (databases are isolated, cannot easily fetch data from a third-party service like stripe), heavy db load (if function does heavy computing (like image processing or intensive text parsing), it consumes core database CPU resources, which can slow down your entire app)

- Edge Functions = typescript/js functions distributed globally across a CDN (like cloudflare workers), they run in data centers physically closest to your mobile user.

	user in arctic triggers function $\rightarrow$ runs in arctic data center $\rightarrow$ it talks to central database $\rightarrow$ returns result

	Pros - low latency for ingress, great dx (dev experience), 3rd-party integrations, offloads db computer
	Cons - network hop problem (main edge function is in arctic, main db hosted in antarctica), cold starts, no direct websockets