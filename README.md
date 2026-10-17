# platform-gitops

This repository is the request folder for a small self-service developer platform. Each file in `requests/` is a request for a piece of cloud infrastructure. Merge a file in and the platform builds it. Delete the file and the platform removes it again.

The platform itself (the portal, the cluster setup and the rules that turn a request into real infrastructure) lives in a separate repository: [self-service-platform](https://github.com/najeebullahii/self-service-platform). This repository only holds the requests, so its history reads as a plain record of who asked for what, and when.

## How a request becomes a bucket

1. A developer fills in a short form in the portal (Backstage).
2. The portal opens a pull request on this repository that adds one file to `requests/`.
3. Someone merges the pull request.
4. Argo CD, running in the cluster, notices the new file. It checks Git about every three minutes by default.
5. Crossplane turns the request into a real Amazon S3 bucket, with public access blocked and tags for the team and for who manages it.
6. To remove the bucket, delete the file. Argo CD notices the file is gone and Crossplane deletes the bucket. A bucket that still has objects in it will not be deleted until it is emptied.

In one test, the bucket was gone between two and two and a half minutes after the file was removed. Most of that time is Argo CD waiting for its next look at Git.

## What a request looks like

```yaml
apiVersion: platform.example.org/v1alpha1
kind: StorageBucket
metadata:
  namespace: default
  name: orders
spec:
  team: orders
```

The only thing a developer has to choose is the team. It must be one of `payments`, `orders` or `platform`. Anything else is refused before it reaches AWS.

## Rules for this repository

- Only request files go in `requests/`. No keys, passwords or tokens, ever. The platform reads its AWS access from the cluster, not from Git.
- One file per request, named after the request.
- `catalog-info.yaml` at the top level is not a request. It describes this repository to the portal's catalog, and Argo CD ignores it because it only watches `requests/`.

## Current state

This was built as a portfolio project. The cluster, the AWS access key and the test buckets were all removed on 17 October 2026, so nothing is watching this folder now. `requests/` is empty on purpose. A placeholder file, `.gitkeep`, keeps the folder in Git, because Argo CD needs the folder to exist.

To run the platform again, follow the steps in the [self-service-platform](https://github.com/najeebullahii/self-service-platform) README.