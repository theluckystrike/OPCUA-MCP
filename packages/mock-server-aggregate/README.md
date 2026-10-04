# Aggregate-capable mock OPC UA server

A small `node-opcua` server used by the end-to-end test suite to exercise
`read_opcua_history` with `aggregate_function` on both MCP servers. Listens on **:4841**
(`opc.tcp://localhost:4841/UA/Aggregate`) unless `AGGREGATE_MOCK_PORT` says
otherwise — the test fixture always sets it to a free port of its own.

## Why a second mock

The main mock (`packages/mock-server`) stays deliberately aggregate-free:

- It advertises **no** aggregate functions, which is what lets the suite assert
  that an aggregate call against it is refused with `capability_not_supported`
  while the argument stays listed
  (`test_an_aggregate_call_against_a_server_without_them_errors_cleanly`).
- It could not serve aggregates anyway — python-opcua's history manager answers
  `ReadProcessedDetails` with `BadNotImplemented`.

This server covers the positive cases. `node-opcua-aggregates` publishes the
standard aggregate function nodes under `ServerCapabilities/AggregateFunctions`
and provides a real `ReadProcessedDetails` implementation, so both MCP servers
compute genuine aggregate values against it.

## Address space

| Node | Node ID | Notes |
|---|---|---|
| `Plant/Temperature` | `ns=1;i=1001` | `Double`, historized, ramps **+1.0 per second** |

The ramp is deliberate and linear so aggregates can be checked arithmetically:
consecutive `Average` buckets must differ by exactly `processing_interval / 1000`.
`AccessHistoryDataCapability` is set, so raw history reads are served too.

## Running

```bash
npm install
npm start          # or: node server.mjs
```

Override the port with `AGGREGATE_MOCK_PORT`. The test fixture starts its own
instance per session on an ephemeral port, so it never adopts — or is disturbed
by — a server belonging to another checkout.
