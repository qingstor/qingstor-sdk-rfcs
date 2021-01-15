# Proposal: Handle metadata

## Background

The official website now contains information about creating object metadata at the following page: [metadata] 

In the SDK, this is basically achieved by setting the x-meta-data parameter to map (dict/hash, slightly different in different languages), then parsing the map and setting the header, but there is no further specification of the key, which are currently two main cases: (upload "x-qs-meta-k": "v" for example)

1. The header key passed by the user is not handled, i.e. the user needs to set {"x-qs-meta-k": "v"} in the x-meta-data map.
  - The advantage is that no additional processing is required by sdk, and if the user input is wrong, it is guaranteed not to be uploaded as metadata.
  - The disadvantage is that the key names are redundant, each key needs to start with "x-qs-meta-", and the semantics are duplicated in the map x-meta-data.

2. Handle the header key from user in SDK, i.e. user only needs to set {"k": "v"} in the x-meta-data map.
  - The advantage is that the information is more explicit and less redundant
  - The disadvantage is that the user may set key as "x-qs-meta-k", so we will have a "x-qs-meta-x-qs-meta-k": "v", such metadata, and the user should not want a metadata like that.1
  
3. Check for the user at the SDK level, as detailed below. 
   (When iterate through the x-meta-data map as example)
  - if the user's key starts with "x-qs-meta-", then we don't need to do any processing, just set the header
  - else manually add the "x-qs-meta-" prefix to it and set the header

## Proposal

As the result after discussion, we will take the second way to handle metadata,
so that only one way was offered for developers, which would make our behaviour
more understandable and unified. 

## Rationale

* Easy for developers to use
* More understandable and behaviour-unified in docs

## Compatibility

All changes should be compatible.

## Implementation

This RFC will be implemented by SDK maintainers.

[metadata]: https://docs.qingcloud.com/qingstor/api/common/metadata#%E5%A6%82%E4%BD%95%E5%88%9B%E5%BB%BA%E5%AF%B9%E8%B1%A1%E5%85%83%E6%95%B0%E6%8D%AE
