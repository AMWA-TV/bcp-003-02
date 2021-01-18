### What does it do?

- Documents best practice for an API server to accept or reject requests depending on what a client is authorized to do.

### Why does it matter?

- A secure control plane is essential.
  - Authorization limits what clients can do to what is allowed.
- These recommendations allow interoperability using widely adopted open technologies.

### How does it work?

- Recommends using [AMWA IS-10 Authorization Specification](https://amwa-tv.github.io/nmos-authorization).
  - This specifies how client provides credentials and gets access tokens.
- Encryption is a prerequisite (see BCP-003-01).
