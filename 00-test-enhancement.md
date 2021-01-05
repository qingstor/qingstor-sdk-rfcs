# Proposal: Test Enhancement

## Background

Obviously, testing is an important part of the project, but our sdks are not tested well which make them don't work as expect from time to time.

Now, we have migrated all build steps from travisCI to Github Actions, and built our own github action runners. We can redesign and set up our tests logic.

## Proposal

So I propose to do following test enhancement:

### Unit Tests

During unit tests, SDK SHOULD NOT: 

- send any HTTP requests;
- use any production values, valid access keys and secret keys, production host and zone names, etc.

SDK should implement following unit test cases:

#### Build Request

> Following <endpoint> means endpoint without protocol and port.

- `ListBuckets` SHOULD send request `GET /` with host `<endpoint>`
- `ListObjects` in virtual host style SHOULD send request `GET /` with host `<bucket-name>.<endpoint>`
- `ListObjects` in path style SHOULD send request `GET /<bucket-name>` with host `<endpoint>`
- `GetObject` in virtual host style SHOULD send request `GET /<object-key>` with host `<bucket-name>.<endpoint>`
- `GetObject` in path style SHOULD send request `GET /<bucket-name>/<object-key>` with host `<endpoint>`
- `PutObject-Copy` in virtual host style SHOULD send request `PUT /<object-key> with host `<bucket-name>.<endpoint>` and header `x-qs-copy-source: <copy-source>`

#### Sign Request

Test Access Key ID: `TEST_ACCESS_KEY_ID`
Test Secret Access Key: `TEST_SECRET_ACCESS_KEY`
Test HTTP `Date`/`x-qs-date`: `Sun, 16 Aug 2015 09:05:00 GMT`

- `GetObject` as anonymous user SHOULD send request without header `authorization`
- `GetObject /test_key` SHOULD send request with:
  - string to sign: `<TODO>`
  - authorization header: `<TODO>`
- `PutObject-Copy /test_key` with `x-qs-copy-source: <copy-source>` SHOULD send request with:
  - string to sign: `<TODO>`
  - authorization header: `<TODO>`

### Integration Tests

Integration tests SHOULD be run on our own github actions runners.

SDK SHOULD implement BDD tests described via [qingstor-sdk-test-scenarios](https://github.com/qingstor/qingstor-sdk-test-scenarios).

Tests will be running in the same bucket allocated statically.

## Compatibility

No business logic changes.

## Implementation

This RFC will be implemented by SDK maintainers.
