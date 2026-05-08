# hf-speedtest

A Python `hf` CLI extension that measures your download bandwidth to HF and its CDN.

![hf-speedtest](assets/hf-speedtest.png)

## Install

Install via the [`hf` CLI extension system](https://huggingface.co/docs/huggingface_hub/guides/cli-extensions):

```bash
hf extensions install julien-c/hf-speedtest
```

Then run:

```bash
hf speedtest
```

To update later:

```bash
hf extensions install julien-c/hf-speedtest --force
```

To remove:

```bash
hf extensions remove speedtest
```

## Example output

```
$ hf speedtest
server location: aws-eu-west-3 (Paris, France)
[##############################] 1808.70 Mbit/s  ( 1453.7 MB in  6.8s)
final: 1808.70 Mbit/s
```

The first line resolves the CDN PoP your test is hitting via the
`x-hf-cdn-pop` response header — it should be your geographically closest edge.

### Underlying providers

| Provider                       | Status    |
| ------------------------------ | --------- |
| AWS                            | supported |
| GCP                            | supported |
| other providers coming soon    | —         |

## How it works

- Opens 8 parallel HTTPS streams to `https://aws.cdn.hf.co/fast/5gb`.
- Ignores the first 1.5 s of bytes so TCP slow-start doesn't skew the result.
- Computes Mbit/s from cumulative bytes received, with a 1.06× factor to
  compensate for TCP/IP/HTTP overhead.
- Auto-shortens the test on fast links — a multi-Gbit link finishes in a few
  seconds instead of running the full 20 s budget.

Knobs are at the top of `src/hf_speedtest/cli.py`:

| Constant            | Default | Equivalent LibreSpeed setting |
| ------------------- | ------- | ----------------------------- |
| `NUM_STREAMS`       | 8       | `xhr_dlMultistream`           |
| `STREAM_STAGGER_MS` | 300     | `xhr_multistreamDelay`        |
| `GRACE_SECONDS`     | 1.5     | `time_dlGraceTime`            |
| `MAX_SECONDS`       | 20.0    | `time_dl_max`                 |
| `OVERHEAD`          | 1.06    | `overheadCompensationFactor`  |

## License

LGPL-3.0 — see [LibreSpeed's license](https://github.com/librespeed/speedtest/blob/master/LICENSE)
for the upstream project this is inspired by.
