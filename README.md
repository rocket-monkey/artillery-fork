# Load Testing ISO, LiveShopping

This repo is used to collect special load-tests for artillery which can be used to load test our ISO platform.

It contains a fork of artillery.io which is used to integrate a custom engine for artillery called `wsgql` - it is an engine to test "Apollo Subscriptions" trough "WebSockets". It is basically the same engine as the original "ws" of artillery, but with the custom "apollo graphql" protocoll on top of it -> see the part of `ws.on('open', function() {` in `core/lib/engine_wsgql.js` 😉

## How to use

1. `cd` into directory `<this-project-dir>/artillery` and run `npm install`
2. still in the directory `<this-project-dir>/artillery` run `npm install -g` to install this special fork globally (remove any existing artillery package by `npm remove -g` first ☝)
3. To test against "graphql websockets", the websocket server (GraphQL.Subscriptions) must be running and you have to know the target url (wss://) - make sure to change the address in any `artillery-tests/load.test.ls.wsgql.yaml` before running!
4. Make sure you know what you do - be ready on the target system to measure the load and be also ready to terminate the test immediately if anything goes wrong `ctrl + c`
5. Now you are able to run the special load test for the engine "wsgql" by changing into `<this-project-dir>` and run `artillery run artillery-tests/load.test.ls.wsgql.yaml`

## Report

* Run `artillery run -o report.json artillery-tests/load.test.ls.wsgql.yaml` to generate a report file of the test!
* Afterwards, run `artillery report report.json` to render the report as nice HTML view!

## Available tests

### Raw HTTP

To test a regular website user there is the test `artillery-tests/load.test.ls.http.yaml`. It fires the initial GET request to load the route `/de/LiveShopping/<id>` - this will include the server-side-rendering, initial GQL requests on the server and so on our ISO/GQL/Monolith/DB/Cache Layer. To further simulate a real browser user, we fire additional POST requests against our graphql endpoint and logging the json results.

With this test, we have a behavior which is very close to a real browser user and generating load troughout the whole application. The only requests we don't make are the ones against `static.digitecgalaxus.ch` which are all driven by Akamai (all static images, scripts and styles), we don't need to test Akamai too in this scenario.

### Websockets GraphQL

As our new LiveShopping solution is integrating an `Apollo GQL Subscription` which is driven trough websockets - we want to test this layer too! This is a feature where we can push new data for live-shopping real-time to connected clients (no reload/F5 triggering needed anymore!).

As this is a new layer for our architecture - we want to make sure we can handle this load too! So we have written a special load-test which is utilizing these websockets too to simulate a full live-shopping experience a user would have.

___
## Final thoughts

To simulate a full load of a live-shopping route, we must run both tests:

* `artillery-tests/load.test.ls.http.yaml` to test the raw HTTP load
* `artillery-tests/load.test.ls.wsgql.yaml` to test the raw WebSocket load

### ArrivalRate / RampUp
The initial section of the artillery tests will include the config about the arrivalRate/rampUp to control how many users/connection over a certain amount of time will arrive during the test.

```yaml
config:
  target: 'some-url'
  phases:
    - duration: 60
      arrivalRate: 5
    - duration: 120
      arrivalRate: 5
      rampTo: 50
    - duration: 600
      arrivalRate: 50

  ...
```
[See artillery.io docs for further information](https://artillery.io/docs/basic-concepts/)
___

