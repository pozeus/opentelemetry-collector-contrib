# Cloudwatch Receiver

| Status                   |           |
| ------------------------ | --------- |
| Stability                | [alpha]   |
| Supported pipeline types | logs      |
| Distributions            | [contrib] |

Receives Cloudwatch events from [AWS Cloudwatch](https://aws.amazon.com/cloudwatch/) via the [AWS SDK for Cloudwatch Logs](https://docs.aws.amazon.com/sdk-for-go/api/service/cloudwatchlogs/)

## Getting Started

This receiver uses the [AWS SDK](https://docs.aws.amazon.com/sdk-for-go/v1/developer-guide/configuring-sdk.html) as mode of authentication, which includes [Profile](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-profiles.html) and [IMDS](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-metadata.html) authentication for EC2 instances.

## Configuration

### Top Level Parameters

| Parameter       | Notes      | type   | Description                                                                                                                                                                                                                                                                       |
| --------------- | ---------- | ------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `region`        | _required_ | string | The AWS recognized region string                                                                                                                                                                                                                                                  |
| `profile`       | _optional_ | string | The AWS profile used to authenticate, if none is specified the default is chosen from the list of profiles                                                                                                                                                                        |
| `imds_endpoint` | _optional_ | string | A way of specifying a custom URL to be used by the EC2 IMDS client to validate the session. If unset, and the environment variable `AWS_EC2_METADATA_SERVICE_ENDPOINT` has a value the client will use the value of the environment variable as the endpoint for operation calls. |
| `logs`          | _optional_ | `Logs` | Configuration for Logs ingestion of this receiver                                                                                                                                                                                                                                 |

### Logs Parameters

| Parameter                | Notes        | type                   | Description                                                                                |
| ------------------------ | ------------ | ---------------------- | ------------------------------------------------------------------------------------------ |
| `poll_interval`          | `default=1m` | duration               | The duration waiting in between requests.                                                  |
| `max_events_per_request` | `default=50` | int                    | The maximum number of events to process per request to Cloudwatch                          |
| `groups`                 | _optional_   | `See Group Parameters` | Configuration for Log Groups, by default all Log Groups and Log Streams will be collected. |

### Group Parameters

`autodiscover` and `named` are ways to control and filter which log groups and log streams which are collected from. They are mutually exclusive and are incompatible to be configured at the same time.

- `autodiscover`
  - `limit`: (optional; default = 50) Limits the number of discovered log groups. This does not limit how large each API call to discover the log groups will be.
  - `prefix`: (optional) A prefix for log groups to limit the number of log groups discovered.
    - if omitted, all log streams up to the limit are collected from
  - `include_linked_accounts`: (optional; default false) A boolean to specify if log groups that is located in other AWS Accounts should be discovered. This assumes that you have configured cross-account access.
  - `streams`: (optional) If `streams` is omitted, then all streams will be attempted to retrieve events from.
    - `names`: A list of full log stream names to filter the discovered log groups to collect from.
    - `prefixes`: A list of prefixes to filter the discovered log groups to collect from.
- `named`
  - This is a map of log group name to stream filtering options
    - `streams`: (optional)
      - `names`: A list of full log stream names to filter the discovered log groups to collect from.
      - `prefixes`: A list of prefixes to filter the discovered log groups to collect from.

#### Autodiscovery Example Configuration

```yaml
awscloudwatch:
  region: us-west-1
  logs:
    poll_interval: 1m
    groups:
      autodiscover:
        limit: 100
        prefix: /aws/eks/
        include_linked_accounts: true
        streams:
          prefixes: [kube-api-controller]
```

#### Named Example

```yaml
awscloudwatch:
  region: us-west-1
  logs:
    poll_interval: 5m
    groups:
      named:
        /aws/eks/dev-0/cluster:
          names: [kube-apiserver-ea9c831555adca1815ae04b87661klasdj]
```

## Sample Configs

This receiver has a number of sample configs for reference.

1. [Default](./testdata/sample-configs/default.yaml)

   - Minimal configuration of the receiver
   - Performs autodiscovery
   - Collects all log groups and log streams

2. [Autodiscover Filtering Log Groups](./testdata/sample-configs/autodiscover-filter-groups.yaml)

   - Performs autodiscovery
   - Only collects log groups matching a prefix
   - Limits the number of discovered Log Groups

3. [Autodiscover Filtering Log Streams](./testdata/sample-configs/autodiscover-filter-streams.yaml)

   - Performs autodiscovery for all Log Groups
   - Filters log streams

4. [Named Groups](./testdata/sample-configs/named-prefix.yaml)

   - Specifies and only collects from the desired Log Groups
   - Does not attempt autodiscovery

5. [Named Groups Filter Log Streams](./testdata/sample-configs/named-prefix-streams.yaml)

   - Specifies the names of the log groups to collect
   - Does not attempt autodiscovery
   - Only collects from log streams matching a prefix

[alpha]: https://github.com/open-telemetry/opentelemetry-collector#alpha
[contrib]: https://github.com/open-telemetry/opentelemetry-collector-releases/tree/main/distributions/otelcol-contrib
