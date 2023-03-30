## Preview

[filename](https://user-images.githubusercontent.com/113217272/227815542-1f0f1af5-5f3b-483a-a26e-e2e73370fdb1.mp4 ':include :type=video controls width=100%')

## Building and Running

### TLDR
```sh
docker run -d -p 5000:5000 --restart=always --name registry registry:2

mkdir nitrox

git clone git@github.com:icebaker/nitrox-builder.git nitrox/nitrox-builder
git clone git@github.com:icebaker/nitrox-core.git nitrox/nitrox-core
git clone git@github.com:icebaker/nitrox-baseline.git nitrox/nitrox-baseline
git clone git@github.com:icebaker/nitrox-ui.git nitrox/nitrox-ui

cd nitrox/nitrox-builder
./build-baseline.sh
./build-ui.sh

cd ../nitrox-baseline
docker compose up -d

cd ../nitrox-ui
docker compose up -d
```
Go to http://localhost:3700

### Detailed

Before building and running Nitrox, make sure that you have [Docker](https://docs.docker.com/get-docker/) and [Docker Compose](https://docs.docker.com/compose/install/linux/) installed on your system.

You also need to ensure that you have a local [Docker Registry](https://docs.docker.com/registry/) running. You can use the following command to start a Docker registry container:

```sh
docker run -d -p 5000:5000 --restart=always --name registry registry:2
```

Next, create a folder named `nitrox` to hold the Nitrox-related repositories:

```sh
mkdir nitrox
```

Clone the Nitrox Builder repository into the `nitrox` folder:
```sh
git clone git@github.com:icebaker/nitrox-builder.git nitrox/nitrox-builder
```

Clone the Nitrox Core repository into the `nitrox` folder:
```sh
git clone git@github.com:icebaker/nitrox-core.git nitrox/nitrox-core
```

Clone the Nitrox Baseline repository into the `nitrox` folder:
```sh
git clone git@github.com:icebaker/nitrox-baseline.git nitrox/nitrox-baseline
```

Clone the Nitrox UI repository into the `nitrox` folder:
```sh
git clone git@github.com:icebaker/nitrox-ui.git nitrox/nitrox-ui
```

Navigate to the Nitrox Builder folder, and build the Nitrox Baseline and Nitrox UI images by executing the commands below:

```sh
cd nitrox/nitrox-builder
./build-baseline.sh
./build-ui.sh
```

Once the baseline images are built, navigate to the Nitrox Baseline folder and start the Nitrox services using Docker Compose:

```sh
cd ../nitrox-baseline
docker compose up -d
```

After starting Nitrox services, navigate to the Nitrox UI folder and launch the Nitrox UI using Docker Compose:

```sh
cd ../nitrox-ui
docker compose up -d
```

Once you start Nitrox using Docker Compose, all the required Nitrox services will start running in the background. You can verify that the services are running by checking the status of the Nitrox containers using the following command:
```sh
docker ps | grep nitrox
```

You should see a list of running Nitrox containers. You can access the Nitrox web interface by navigating to http://localhost:3700 in your web browser.

## Connecting Your Node

Once Nitrox is up and running, you can add a connection to your Node through the Nitrox UI at http://localhost:3700/connections. We recommend reading the [Connecting section from Lighstorm](https://icebaker.github.io/lighstorm/#/README?id=connecting), as it provides insights into how connections work.

### Preparing Your Node

You need to ensure that your node accept connections from the Docker Host IP (`172.17.0.1`). Otherwise, you may receive the following error when trying to connect:

```text
Peer name 172.17.0.1 is not in peer certificate.
```

To connect to an LND node through a Docker container or remote host, you need to adjust your certificate settings. Follow these steps:

1. Stop your LND node.

2. Remove or backup existing certificate files (`tls.cert` and `tls.key`) in the LND directory.

For example, if you are using [Umbrel](https://umbrel.com), your [lnd](https://github.com/lightningnetwork/lnd) data resides at `/home/your-user-name/umbrel/app-data/lightning/data/lnd/data`. Another common location is `/home/your-user-name/.lnd`.

3. Modify `lnd.conf` to include the relevant `tlsextraip` and/or `tlsextradomain` settings:

Option A: Accept any IP or domain (⚠ **Warning:** high security risk):

```conf
tlsextraip=0.0.0.0
```

Option B: Accept only your Docker host (`172.17.0.1`):
```conf
tlsextraip=172.17.0.1
```

Option C: Accept a specific remote domain and host:
```config
tlsextraip=<your_remote_host_ip>
tlsextradomain=<your_domain_name>
```

4. Save and restart your LND node. New `tls.cert` and `tls.key` files will be generated.

Choose the option that best suits your needs and environment while considering security implications.


### Methods

#### File Paths

To connect using file paths, you must first provide volumes to enable the connection.

Navigate to `nitrox/nitrox-baseline/docker-compose.yaml` and locate the `connector` section:


```yml
  connector:
    image: localhost:5000/nitrox-connector
    environment:
      NITROX_DISCOVERY: discovery:3000
      NITROX_SERVICE: nitrox/connector
      NITROX_ENVIRONMENT: production
      NITROX_HOST: connector
      NITROX_PORT: 3000
    volumes:
      - ./data/connector:/nitrox/app/service/nitrox-connector/data
    depends_on:
      - discovery
```

For example, if you are using [Umbrel](https://umbrel.com), your [lnd](https://github.com/lightningnetwork/lnd) data resides at `/home/your-user-name/umbrel/app-data/lightning/data/lnd/data`. Another common location is `/home/your-user-name/.lnd`. To make it accessible inside the connector, you need to:

```yml
  connector:
    image: localhost:5000/nitrox-connector
    environment:
      NITROX_DISCOVERY: discovery:3000
      NITROX_SERVICE: nitrox/connector
      NITROX_ENVIRONMENT: production
      NITROX_HOST: connector
      NITROX_PORT: 3000
    volumes:
      - ./data/connector:/nitrox/app/service/nitrox-connector/data
      - /home/your-user-name/umbrel/app-data/lightning/data/lnd:/lnd:ro
    depends_on:
      - discovery
```

Run `docker-compose up -d` inside `nitrox/nitrox-baseline` to update the containers. Now, you can add a new connection through the interface at http://localhost:3700/connections. Click on "Create Connection," enter a name for your connection, select the "Paths" tab, and add the following data:

- Address: `172.17.0.1:10009`
- Certificate Path: `/lnd/tls.cert`
- Macaroon Path: `/lnd/data/chain/bitcoin/mainnet/admin.macaroon`

You can't use your localhost IP (`127.0.0.1`) because the container won't be able to reach it. Instead, you need to use the Docker Host IP (e.g. `172.17.0.1`) to make it work.

Alternatively, you can set environment variables in the compose file instead of the interface, following [Lighstorm's nomenclature](https://icebaker.github.io/lighstorm/#/README?id=environment-variables):

```yml
  connector:
    image: localhost:5000/nitrox-connector
    environment:
      NITROX_DISCOVERY: discovery:3000
      NITROX_SERVICE: nitrox/connector
      NITROX_ENVIRONMENT: production
      NITROX_HOST: connector
      NITROX_PORT: 3000
      LIGHSTORM_LND_ADDRESS: 172.17.0.1:10009
      LIGHSTORM_LND_CERTIFICATE_PATH: /lnd/tls.cert
      LIGHSTORM_LND_MACAROON_PATH: /lnd/data/chain/bitcoin/mainnet/admin.macaroon
    volumes:
      - ./data/connector:/nitrox/app/service/nitrox-connector/data
      - /home/your-user-name/umbrel/app-data/lightning/data/lnd:/lnd:ro
    depends_on:
      - discovery
```

Run `docker-compose up -d` inside `nitrox/nitrox-baseline` to update the containers.

#### Credentials

If you don't want to create Docker volumes or rely on files, you can simply get your node credentials and use them to create the connection.

First, get the credentials for the certificate and macaroon. One way of doing this is through the `base64` command:

```sh
base64 -w0 ~/.lnd/tls.cert
base64 -w0 ~/.lnd/data/chain/bitcoin/mainnet/admin.macaroon
```

If you are using [Umbrel](https://umbrel.com):
```sh
base64 -w0 ~/umbrel/app-data/lightning/data/lnd/tls.cert
base64 -w0 ~/umbrel/app-data/lightning/data/lnd/data/chain/bitcoin/mainnet/admin.macaroon
```

⚠ **Warning:** Be careful not to include the dangling `%` at the end of the string when copying it.

Then, go to http://localhost:3700/connections, click on "Create Connection", choose a name for your connection, and select the "Encoded" tab. Your node address is usually `172.17.0.1:10009`. Paste the certificate and macaroon data generated using `base64`.

Alternatively, you can set environment variables in the compose file instead of using the Nitrox UI interface. This approach follows [Lighstorm's nomenclature](https://icebaker.github.io/lighstorm/#/README?id=environment-variables).

To set the environment variables, navigate to `nitrox/nitrox-baseline/docker-compose.yaml` and locate the `connector` section. Add the following environment variables to configure your Lightning Node connection:

```yml
  connector:
    image: localhost:5000/nitrox-connector
    environment:
      NITROX_DISCOVERY: discovery:3000
      NITROX_SERVICE: nitrox/connector
      NITROX_ENVIRONMENT: production
      NITROX_HOST: connector
      NITROX_PORT: 3000
      LIGHSTORM_LND_ADDRESS: 172.17.0.1:10009
      LIGHSTORM_LND_CERTIFICATE: "LS0tLS1CRU...UtLS0tLQo="
      LIGHSTORM_LND_MACAROON: "AgEDbG5kAv...inv45ukJ4="
    volumes:
      - ./data/connector:/nitrox/app/service/nitrox-connector/data
    depends_on:
      - discovery
```

Run `docker-compose up -d` inside `nitrox/nitrox-baseline` to update the containers.

#### Regtest

If you're developing or testing the Lightning Network, you can use [Polar](https://lightningpolar.com) to create nodes. Once you have your nodes set up, connecting them to Nitrox should be intuitive at http://localhost:3700/connections.


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
