# Task 006 Provenance

## Image manifest digests

Remote manifests were fetched via `docker manifest inspect` in Task 005. Config digests:

| image | config digest | compressed layer bytes |
|---|---|---:|
| vul | `sha256:09112d6823af9ed42134a8ea8fd5d23576c7151a71a40f406e702aea4835b690` | 1,865,010,767 |
| fix | `sha256:3c3013128c3c3da766578d894b28dc30f37023b40093eb080937292a6b019c2f` | 1,107,125,362 |

Full registry manifest digests were not separately recorded; only the config digest is recoverable from Task 005 logs. Layer digests were all verified SHA-256 during download (script `/tmp/task005_pull.py` rejects mismatched blobs).

Note: `docker load` imports an archive built from verified blobs, so the local `RepoDigest` shows `<none>`; the config digest above is the canonical identity and matches the registry manifest.

## Image IDs

| image | ID |
|---|---|
| vul | `sha256:09112d6823af9ed42134a8ea8fd5d23576c7151a71a40f406e702aea4835b690` |
| fix | `sha256:3c3013128c3c3da766578d894b28dc30f37023b40093eb080937292a6b019c2f` |

## Docker-load archive

Built via `/tmp/task005_pull.py`: downloaded blobs from registry API (via local proxy `127.0.0.1:7890`), SHA-256 verified per blob, assembled into legacy Docker-load tar archives, imported with `docker load`.

## Run commands

```bash
docker run --rm --network none --cpus 2 --memory 8g --pids-limit 512 \
  --security-opt no-new-privileges --platform linux/amd64 \
  n132/arvo:42486052-vul arvo

docker run --rm --network none --cpus 2 --memory 8g --pids-limit 512 \
  --security-opt no-new-privileges --platform linux/amd64 \
  n132/arvo:42486052-fix arvo
```

Network disabled, CPU 2, memory 8g, PID limit 512, no-new-privileges: all confirmed in Task 005 command log.
