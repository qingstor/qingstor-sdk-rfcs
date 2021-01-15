# Proposal: Handle metadata

## Background

The official website now contains information about creating object metadata at the following page: [metadata] 

In the SDK, this is basically achieved by setting the `x-meta-data` parameter to map (dict/hash, slightly different in different languages), 
then parsing the map and setting the header, but there is no further specification of the key.

So I propose a way to parse the `x-meta-data` map into http header.

## Proposal

In each SDK, the `x-meta-data` map should be parsed into http header when build the request.
The developer only needs to set `{"k": "v"}` in the `x-meta-data` map, and the prefix `x-qs-meta-`
will be added when build request. So the parsed http header will be `{"x-qs-meta-k": "v"}`.

On one hand, this ensures all k-v pairs available for QingStor API, 
more explicit and less redundant information for developers. 

On the other hand, only one way was offered for developers, which would make our behaviour 
more understandable and unified.

## Rationale

### Other plans not applied 
(upload "x-qs-meta-k": "v" for example)

1. The header key passed by the user will not be handled, i.e. the user needs to set {"x-qs-meta-k": "v"} in the x-meta-data map.
  - The advantage is that no additional processing is required by sdk, and if the user input is wrong, it is guaranteed not to be uploaded as metadata.
  - The disadvantage is that the key names are redundant, each key needs to start with "x-qs-meta-", and the semantics are duplicated in the map x-meta-data.

2. Check for the user at the SDK level, as detailed below. 
  - When iterate through the x-meta-data map as example
    - if the user's key starts with "x-qs-meta-", then we don't need to do any processing, just set the header
    - else manually add the "x-qs-meta-" prefix to it and set the header
  - The disadvantage is that it is not a unified behaviour, sometimes it is not good to offer more than one choice. 
 

## Compatibility

All changes should be compatible.

## Implementation

This RFC will be implemented by SDK maintainers.

[metadata]: https://docs.qingcloud.com/qingstor/api/common/metadata#%E5%A6%82%E4%BD%95%E5%88%9B%E5%BB%BA%E5%AF%B9%E8%B1%A1%E5%85%83%E6%95%B0%E6%8D%AE
