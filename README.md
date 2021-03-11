# signed-container

A signed container example, with cosign.

## How It Works

This container image is built and pushed to Docker Hub at `dlorenc/signed-container` every time a commit is merged to main.
The containers are tagged with the git commit the container was built at.
For example: `dlorenc/signed-container:a5525b0df6fb6683cd2a01e01dfb0c92252b1b65` was built at the commit `a5525b0df6fb6683cd2a01e01dfb0c92252b1b65`.

The public key that can be used to verify each container image is stored right in the git repo, in the file `cosign.pub`.
The private key is also stored right here (encrypted!) in `cosign.key`.
The GitHub action is configured with a secret containing the password for this key (`CosignPassword`).

## Key Rotation

With this flow (committing keys into the repo), rotating keys is as simple as generating and commiting a new key pair.
Each build contains a (signed) reference back to the commit (with public key)!

You can verify the entire history by checking out each release tag and verifying the releases against the public keys
present in the repo **at that time**.

## Verifying

You can do a simple verification with:

```shell
$ cosign verify -key cosign.pub dlorenc/signed-container:v0.0.1 | jq .
The following checks were performed on these signatures:
  - The cosign claims were validated
  - The signatures were verified against the specified public key
{
  "Critical": {
    "Identity": {
      "docker-reference": ""
    },
    "Image": {
      "Docker-manifest-digest": "sha256:b5d83b473776186d1b3670433d759c786ec413aacf46c5fad606b11043d0368d"
    },
    "Type": "cosign container signature"
  },
  "Optional": {
    "git_sha": "a5525b0df6fb6683cd2a01e01dfb0c92252b1b65"
  }
}
```

This tells you that the container was signed with the private key in this repo.

### CI Builds

The continuous CI builds are signed with an extra annotation including the git commit they were built at.
This can be verified with:

```
$ cosign verify -a  git_sha=$(git rev-parse HEAD) -key cosign.pub dlorenc/signed-container:v0.0.1 | jq .
The following checks were performed on these signatures:
  - The specified annotations were verified.
  - The cosign claims were validated
  - The signatures were verified against the specified public key
{
  "Critical": {
    "Identity": {
      "docker-reference": ""
    },
    "Image": {
      "Docker-manifest-digest": "sha256:b5d83b473776186d1b3670433d759c786ec413aacf46c5fad606b11043d0368d"
    },
    "Type": "cosign container signature"
  },
  "Optional": {
    "git_sha": "a5525b0df6fb6683cd2a01e01dfb0c92252b1b65"
  }
}
```

### Releases

The release builds are signed with another extra annotation including the git tag (in addition to the commit):

```
$ cosign verify -a  git_sha=$(git rev-parse v0.0.1) -key cosign.pub dlorenc/signed-container:v0.0.1 | jq .
The following checks were performed on these signatures:
  - The specified annotations were verified.
  - The cosign claims were validated
  - The signatures were verified against the specified public key
{
  "Critical": {
    "Identity": {
      "docker-reference": ""
    },
    "Image": {
      "Docker-manifest-digest": "sha256:b5d83b473776186d1b3670433d759c786ec413aacf46c5fad606b11043d0368d"
    },
    "Type": "cosign container signature"
  },
  "Optional": {
    "git_sha": "a5525b0df6fb6683cd2a01e01dfb0c92252b1b65",
    "git_tag": "v0.0.1"
  }
}
```
