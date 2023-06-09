# Nitrox

A delightful Lightning Network experience.

https://user-images.githubusercontent.com/113217272/227815542-1f0f1af5-5f3b-483a-a26e-e2e73370fdb1.mp4

## Index

- [Installing](https://icebaker.github.io/nitrox)
- [Documentation](https://icebaker.github.io/nitrox)
- [Security](#security)
- [Open Source and Transparency](#open-source-and-transparency)

## Security

As both the Lightning Network and Nitrox are currently in [**beta**](https://en.wikipedia.org/wiki/Software_release_life_cycle#Beta), it is important for you to remember **not to put bitcoins on the network that you are not willing to lose**.

While using Nitrox in its current beta version, please keep in mind the following security considerations:

- [Network](#network)
- [Third-Party Dependencies](#third-party-dependencies)
- [File System](#file-system)
- [Signature Verifications](#signature-verifications)

### Network
Nitrox assumes that your local network and computer are secure. As a result, local network communication is not encrypted, using plain text HTTP, and there are no passwords required to access interfaces and APIs.

Additionally, all services and APIs are exposed under the Docker Host Network, which means your computer's local host.

In summary, Nitrox's security relies on the security of your local network and computer.

### Third-Party Dependencies

Nitrox depends on third-party open-source software, and we cannot fully examine the potential security implications of each component.

A non-exhaustive list of examples for third-party dependencies includes:

- Infrastructure: [memcached](https://memcached.org), [BadgerDB](https://dgraph.io/docs/badger/), [Redpanda](https://redpanda.com)
- Node.js: [Svelte](https://svelte.dev), [Nano ID](https://github.com/ai/nanoid), [Vite](https://vitejs.dev), and others.
- Ruby: [Puma](https://puma.io), [Roda](https://roda.jeremyevans.net), [Faraday](https://lostisland.github.io/faraday/), [Zache](https://github.com/yegor256/zache), [rdkafka](https://github.com/appsignal/rdkafka-ruby), [Dalli](https://github.com/petergoldstein/dalli), and others.
- Go: [Fiber](https://docs.gofiber.io), [godotenv](https://pkg.go.dev/github.com/joho/godotenv?utm_source=godoc), and others.

### File System

The containers and services create local, unencrypted volumes on your file system. The security of these volumes depends on the security of your computer. Unauthorized access to your computer could compromise your data.

### Signature Verifications

There is no signature verification for Docker Images or GitHub clones, although all maintainers' commits are signed.

Both [_Docker, Inc._](https://en.wikipedia.org/wiki/Docker,_Inc.) and [_GitHub, Inc._](https://en.wikipedia.org/wiki/GitHub) are private companies over which we have no control, other than trusting their security measures and good faith.

Although it is possible to perform signature verifications for repositories and Docker images, Nitrox does not currently provide documentation or tools to guide users through this process.

## Open Source and Transparency

Nitrox is an open-source Lightning Node Management Application with an [MIT license](https://github.com/icebaker/nitrox/blob/main/LICENSE), which means that anyone can use, modify, and build upon the project for any purpose.

The main version of Nitrox may include optional paid features or services, such as submarine swaps, liquidity provision, and routing optimization, a common practice in node management applications. Services from the project owner may be prioritized in the interface, always with transparency to users. Multiple providers can coexist, allowing users the freedom to choose their preferred provider.

When contributing to the project, please be aware of the owner's control over service offerings. Your contributions are important to the project's growth and success. We appreciate your involvement in helping shape the future of Nitrox and enhancing the Lightning Network experience for everyone within the open-source community.
