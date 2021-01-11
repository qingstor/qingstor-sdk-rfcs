# Proposal: Handle metadata

## Background

The official website now contains information about creating object metadata at the following page: [metadata] 

In the SDK, this is basically achieved by setting the x-meta-data parameter to map (dict/hash, slightly different in different languages), then parsing the map and setting the header, but there is no further specification of the key, which are currently two main cases: (upload "x-meta-data-k": "v" for example)

1. The header key passed by the user is not handled, i.e. the user needs to set {"x-meta-data-k": "v"} in the x-meta-data map.
  - The advantage is that no additional processing is required by sdk, and if the user input is wrong, it is guaranteed not to be uploaded as metadata.
  - The disadvantage is that the key names are redundant, each key needs to start with "x-meta-data-", and the semantics are duplicated in the map x-meta-data.

2. Handle the header key from user in SDK, i.e. user only needs to set {"k": "v"} in the x-meta-data map.
  - The advantage is that the information is more explicit and less redundant
  - The disadvantage is that the user may set key as "x-meta-data-k", so we will have a "x-meta-data-x-meta-data-k": "v", such metadata, and the user should not want a metadata like that.1

## Proposal

So I propose that we should think one more step further, and check for the user at the SDK level, as detailed below.
When iterate through the x-meta-data map.
  - if the user's key starts with "x-meta-data-", then we don't need to do any processing, just set the header
  - else manually add the "x-meta-data-" prefix to it and set the header

This is compatible with the above two cases, even if the user really wants to set a metadata key as "x-meta-data-x-meta-data-k", he only needs to pass this key.

## Rationale

## Compatibility

All changes should be compatible.

## Implementation

This RFC will be implemented by SDK maintainers.

[metadata]: https://docs.qingcloud.com/qingstor/api/common/metadata#%E5%A6%82%E4%BD%95%E5%88%9B%E5%BB%BA%E5%AF%B9%E8%B1%A1%E5%85%83%E6%95%B0%E6%8D%AE
