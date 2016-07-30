As it turns out, there's many different ways in which you can share messages between a `ServiceWorker` and the web pages it controls.

- A client may want to [message a `ServiceWorker`][1], asking something or notifying about something _-- this is "unicast" (1-to-1)_
- A `ServiceWorker` may want to send a [reply to a client][2] _-- unicast_
- A `ServiceWorker` may want to send [an update to every client][3] under its control _-- a "broadcast" (1-to-many) message_
- A `ServiceWorker` may want to send [an update][4] to the client where a `fetch` request originated _-- unicast_

[1]: #messaging-the-serviceworker
[2]: #getting-replies-from-the-serviceworker
[3]: #broadcasting-from-a-serviceworker-to-every-client
[4]: #dual-channeling-fetch-requests
