# Proposal: Config Refactor

## Background

For historical reasons, our SDK has the following problems:

- SDK's config path is hardcoded to `~/.config/qingstor.yaml`, if not exist, SDK will create automatically.
- There is not way for user to change SDK's config path during runtime. This enforces all tools to implement their own
  SDK loading logic.
- SDK's assume all user have access key pairs which forbids anonymous API call.
- SDK use virtual host style to check bucket but use path style for API call.
- SDK can't control IPv4/IPv6 logic

## Proposal

So I propose to do following config refactors:

### Environment First Config

Instead of hardcoded `~/.config/qingstor.yaml`, we will read config path from system environment
variable `QINGSTOR_CONFIG_PATH`.

If `QINGSTOR_CONFIG_PATH` is not exists, we will fallback to `~/.config/qingstor.yaml`.

And all SDKs should allow reading config from envs:

```shell
export QINGSTOR_ACCESS_KEY_ID=example_access_key_id
export QINGSTOR_SECRET_ACCESS_KEY=example_secret_access_key
export QINGSTOR_XYZ=example_xyz
```

For every config, we should check via following order:

- The specific env: `QINGSTOR_ACCESS_KEY_ID`
- Config loading from `QINGSTOR_CONFIG_PATH`
- If no default value, keep empty, or use the default value.

All qingstor native tools could use the same logic here.

### Merge Host, Port, Protocol into Endpoint

Instead of use split `host`, `port`, `protocol` config values, we will introduce a new config: `endpoint`.

```yaml
host: qingstor.com
port: 443
protocol: https
```

equals to

```yaml
endpoint: https://qingstor.com:443
```

> Especially for `http://xxx.com:80` && `https://xxx.com:443`, the tailing port number could be omitted.

If user has set `endpoint`, `host`, `port` and `protocol` should be ignored with a warning printed for user.

The default value for `endpoint` is `https://qingstor.com`

### Virtual Host and Path Style support

SDK should support both `virtual host style` and `path style`.

We will introduce a new config: `enable_virtual_host_style`. If set to `true`, SDK will use `virtual host style` for all
API calls.

By default, `enable_virtual_host_style` will be `false`.

The related env key is `QINGSTOR_ENABLE_VIRTUAL_HOST_STYLE`.

### IPv4 / IPv6 support

SDK should support both `ipv6` and `ipv4`.

We will introduce a new config: `enable_dual_stack`. If set to `true`, SDK will enable IPv6 support.

By default, `enable_dual_stack` will be `false`.

The related env key is `QINGSTOR_ENABLE_DUAL_STACK`.

### Anonymous API call support

If either `access_key_id` or `secret_access_key` is empty, SDK should return an initiation error. If
both `access_key_id` and `secret_access_key` is empty, SDK should ignore sign logic and send anonymous API directly.

## Rationale

### Why introduce Environment First Config?

Our current design supports only two kinds: passing directly into the code and reading from the default path, but this
is not enough.

For example, in container and Serverless runtime environments, developers don't have the ability to control the runtime
environment and expose the key directly, so the more common way to configure it is to pass it in via environment
variables. If our SDK did not provide such a capability, then all developers using our SDK would need to handle the
logic themselves, which is unnecessary and could be provided by us. In addition to environment variables, we will
subsequently support the acquisition of keys from IAM, etc.

In addition, we have the ability to provide such a mechanism, which is standard practice for major cloud providers.

For example:

- AWS: <https://docs.aws.amazon.com/sdk-for-javascript/v2/developer-guide/loading-node-credentials-environment.html>
  provides `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`
- Google Cloud: <https://cloud.google.com/docs/authentication/production#auth-cloud-implicit-go>
  provides `GOOGLE_APPLICATION_CREDENTIALS`

In particular, we need to support many private cloud clients, so we need to expose other configuration items in addition
to credentials.

## Compatibility

All changes should be compatible.

For all old behavior, we will print a warning message for user and mark related functions as deprecated.

After two minor versions, we can remove all warning messages and deprecated functions, and maintain upgrade docs.

## Implementation

This RFC will be implemented by SDK maintainers.
