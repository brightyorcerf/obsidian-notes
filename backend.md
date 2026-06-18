#### RPCs vs edge functions

When building an app using a backend-as-a-service like Supabase, need to choose between RPCs (remote procedure calls) and Edge Functions to handle your backend logic. Difference is _where_ they live, _how_ they run, and _what_ they are best suited for.

- RPCs = Custom SQL function written directly inside your PostgreSQL database. App calls this function via an API client, executing the database code seamlessly.

	sends a request $\rightarrow$ API gateway routes it directly to the database $\rightarrow$ the database executes the SQL logic and returns the data.

	Pros - blazing fast performance, ACID transactions, strong security (RLS)
	Cons - sql only, no external api interactions (databases are isolated, cannot easily fetch data from a third-party service like stripe), heavy db load (if function does heavy computing (like image processing or intensive text parsing), it consumes core database CPU resources, which can slow down your entire app)

- Edge Functions = typescript/js functions distributed globally across a CDN (like cloudflare workers), they run in data centers physically closest to your mobile user.

	user in arctic triggers function $\rightarrow$ runs in arctic data center $\rightarrow$ it talks to central database $\rightarrow$ returns result
    

	Pros - low latency for ingress, great dx (dev experience), 3rd-party integrations, offloads db computer
	Cons - network hop problem (main edge function is in arctic, main db hosted in antarctica), cold starts, no direct websockets