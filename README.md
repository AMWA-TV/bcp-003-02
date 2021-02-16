# AMWA BCP-003-02: Authorization in NMOS Systems \[Work In Progress\]

[![Lint Status](https://github.com/AMWA-TV/nmos-authorization-practice/workflows/Lint/badge.svg)](https://github.com/AMWA-TV/nmos-authorization-practice/actions?query=workflow%3ALint)
[![Render Status](https://github.com/AMWA-TV/nmos-authorization-practice/workflows/Render/badge.svg)](https://github.com/AMWA-TV/nmos-authorization-practice/actions?query=workflow%3ARender)

<!-- INTRO-START -->

### What does it do?

- Documents best practice for an API server to accept or reject requests depending on what a client is authorized to do.

### Why does it matter?

- A secure control plane is essential.
  - Authorization limits what clients can do to what is allowed.
- These recommendations allow interoperability using widely adopted open technologies.

### How does it work?

- Recommends using [AMWA IS-10 Authorization Specification](https://specs.amwa.tv/is-10)
  - This specifies how client provides credentials and gets access tokens.
- Encryption is a prerequisite (see BCP-003-01).

<!-- INTRO-END -->

## Getting started

There is more information about the NMOS Specifications and their GitHub repos at <https://specs.amwa.tv/nmos>.
