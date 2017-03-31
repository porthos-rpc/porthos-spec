# Porthos

A language-agnostic RPC library that operates over AMQP.

## Goal

Provide language-agnostic RPC libraries to build distributed systems. Porthos is intended to be a low latency, lightweight, AMQP-based set of libraries without way too many features that only few people care about.

## Benefits over HTTP-based API's

Porthos (pretty much due to the AMQP features) provide some benefits over HTTP-based APIs:
- **Implicit Load Balancing**: Every porthos service declares a queue in the broker. You can have as many instances of the same service as you desire. The AMQP server users round-robin to deliver message to queue consumers (the services).
- **Implicit Service Discovery**: Clients only need to know the location of the broker (AMQP server). The broker is then responsible for delivering messages to the services (that act as consumers of a given queue).
- **Implicit Retry**: Porthos uses the `ack-late` approach. If your remote procedure cannot finish gracefully, the request message goes back to the queue to be redelivered to another service instance.
- **Low overhead/latency**: AMQP is a binary protocol over TCP, it's designed to be fast and compact. Porthos client holds a TPC connection with the broker and all requests are sen't through it.

## Server

- Declares one queue for incoming requests. The queue takes the name of the server.
- The AMQP message for the incoming request must contain:
	- A Header `X-Method` to specify the request method to be invoked.
	- The property `Content-Type` set accordingly.
	- The property `CorrelationId` set accordingly if the client needs to handle the responde.
	- The property `ReplyTo` set with the name of the response queue (if the method need to publish a response back to the client).
	- The Body with the payload of the request (if any).
- The AMQP message for the outgoing response must contain:
	- The Headers returned by the server handler.
	- The property `Content-Type` returned by the server handler.
	- The property `CorrelationId` set with the same value sent in the `CorrelationId` of the AMQP request message.
	- The Body of the response returned by the server handler (if any).

## Client

- Declares one anonymous queue for incoming responses.
- Takes the service name (that is also the request queue name), that is used to send all request messages to.
- Needs to correlate requests and response through the `CorrelationId` sent in the Request and returned in the Response.
