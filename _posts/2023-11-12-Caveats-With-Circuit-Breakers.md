---
title: "Caveats with circuit breakers"
date: 2023-11-12
---

Reference : https://martinfowler.com/bliki/CircuitBreaker.html

The TLDR version of the above is that a circuit breaker is a mechanism by which a client "short circuits" calls to an endpoint based on a number of "failures" seen. This comes from the idea that further retries on a total failure (i.e. in which a service behind the endpoint is hard down) will only aggrevate the problem for the service.

This works great for simple one size fits all endpoints, where you fire requests and move on, but does not always work for clients that need to be "context aware" of where they send requests.

For instance, I was working on a circuit breaker design for Kinesis Streams and immediately realized its limitations. Kinesis streams have shards and the client cannot directly state which shard to talk to (all it can do is construct a partition key/explicit hash key and then send it over to the kinesis frontend https://docs.aws.amazon.com/kinesis/latest/APIReference/API_PutRecord.html#Streams-PutRecord-request-ExplicitHashKey).

This means that if I send a request over to Kinesis with a given hash key, the only thing i can reliably tell is that for that given hash key, the shard it is routing to is either overwhelmed or working fine. So if I count that towards the counter on which to trip my circuit breaker on, I can potentially starve requests intended for other shards (this stems from the fact that the client is not directly aware which shard the partition key mapped to).

This is one example. Other services like Dynamo DB has similar nuances, in which partition keys are mapped to partitions in a dynamo db and one can starve requests intended for one partition that isnt starved based on a circuit trigger from failures on a hot partition.




