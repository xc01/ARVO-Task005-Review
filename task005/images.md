# Task 005 Images

| image | compressed bytes | local image ID | local size | architecture |
|---|---:|---|---:|---|
| `n132/arvo:42486052-vul` | 1,865,010,767 | `sha256:09112d6823af9ed42134a8ea8fd5d23576c7151a71a40f406e702aea4835b690` | 4.37 GB | linux/amd64 |
| `n132/arvo:42486052-fix` | 1,107,125,362 | `sha256:3c3013128c3c3da766578d894b28dc30f37023b40093eb080937292a6b019c2f` | 2.37 GB | linux/amd64 |

Remote manifests were fetched successfully with `docker manifest inspect`. The daemon's configured proxy `http://172.17.0.1:17891` refused connections, so `docker pull` could not complete. Because the manifest and blob digests were verified locally, the images were assembled into Docker-load archives and imported without modifying the daemon. Local digest tags were unavailable after load, and no attempt was made to modify or delete existing images.
