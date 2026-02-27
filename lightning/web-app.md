---
title: Web app
nav_order: 30
parent: Lightning
---

# 3.3 Web app: ThunderHub

[ThunderHub](https://thunderhub.io/) is an open-source LND node manager where you can manage and monitor your node on any device or browser. It allows you to take control of the Lightning Network with a simple and intuitive UX and the most up-to-date tech stack.

<figure><img src="../.gitbook/assets/thunderhub_logo.png" alt=""><figcaption></figcaption></figure>

## Requirements

* [Bitcoin Core](../bitcoin/bitcoin/bitcoin-client.md)
* [LND](lightning-client.md)
* Others
  * [Node + NPM](../bonus/system/nodejs-npm.md)

## Preparations

### Check Node + NPM

* With the user `admin`, check the Node version

```sh
node -v
```

**Example** of expected output:

```
v16.14.2
```

* Check the NPM version

```sh
npm -v
```

**Example** of expected output:

```
8.19.3
```

{% hint style="info" %}
-> If the "`node -v"` output is **`>=24`**, you can move to the next section.

-> If Node.js is not installed (`-bash: /usr/bin/node: No such file or directory`), follow this [Node + NPM bonus guide](../bonus/system/nodejs-npm.md) to install it
{% endhint %}

### Reverse proxy & Firewall

In the security [section](../index-1/security.md#prepare-nginx-reverse-proxy), we set up Nginx as a reverse proxy. Now we can add the ThunderHub configuration.

Enable the Nginx reverse proxy to route external encrypted HTTPS traffic internally to ThunderHub. The `error_page 497` directive instructs browsers that send HTTP requests to resend them over HTTPS.

* With user `admin`, create the reverse proxy configuration

```sh
sudo nano /etc/nginx/sites-available/thunderhub-reverse-proxy.conf
```

* Paste the following complete configuration. Save and exit

```nginx
server {
  listen 4002 ssl;
  error_page 497 =301 https://$host:$server_port$request_uri;

  location / {
    proxy_pass http://127.0.0.1:3001;
  }
}
```

* Create the symbolic link that points to the directory `sites-enabled`

{% code overflow="wrap" %}
```bash
sudo ln -s /etc/nginx/sites-available/thunderhub-reverse-proxy.conf /etc/nginx/sites-enabled/
```
{% endcode %}

* Test Nginx configuration

```sh
sudo nginx -t
```

Expected output:

```
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

* Reload the Nginx configuration to apply changes

```bash
sudo systemctl reload nginx
```

* Configure the firewall to allow incoming HTTP requests from anywhere to the web server

```sh
sudo ufw allow 4002/tcp comment 'allow ThunderHub SSL from anywhere'
```

## Installation

### Create the thunderhub user & group

We do not want to run Thunderhub code alongside `bitcoind` and `lnd` because of security reasons. For that, we will create a separate user and run the code as the new user. We will install Thunderhub in the home directory since it doesn't need too much space.

* Create a new `thunderhub` user and group

```sh
sudo adduser --disabled-password --gecos "" thunderhub
```

* Add `thunderhub` user to the `lnd` group to allow the user `thunderhub` reading the `admin.macaroon` and `tls.cert` files

```sh
sudo adduser thunderhub lnd
```

* Change to the `thunderhub` user

```bash
sudo su - thunderhub
```

* Set a temporary version environment variable for the installation

```bash
VERSION=0.15.1
```

* Import the GPG key of the developer

```bash
curl https://github.com/apotdevin.gpg | gpg --import
```

* Download the source code directly from GitHub, select the latest release branch associated with it, and go to the `thunderhub` folder

{% code overflow="wrap" %}
```sh
git clone --branch v$VERSION https://github.com/apotdevin/thunderhub.git && cd thunderhub
```
{% endcode %}

* Verify the release

```bash
git verify-commit v$VERSION
```

**Example** of expected output:

```
gpg: Signature made Fri May 26 16:56:42 2023 CEST
gpg:                using RSA key 3C8A01A8344B66E7875CE5534403F1DFBE779457
gpg: Good signature from "Anthony Potdevin <potdevin.anthony@gmail.com>" [unknown]
gpg: WARNING: This key is not certified with a trusted signature!
gpg:          There is no indication that the signature belongs to the owner.
Primary key fingerprint: 3C8A 01A8 344B 66E7 875C  E553 4403 F1DF BE77 9457
```

* Install all dependencies and the necessary modules using NPM

{% hint style="warning" %}
**Not to run** the `npm audit fix` command, which could break the original code!!
{% endhint %}

```sh
npm install
```

<details>

<summary><strong>Example</strong> of expected output ⬇️</summary>

```
npm warn deprecated @types/cron@2.4.0: This is a stub types definition. cron provides its own type definitions, so you do not need this installed.
npm warn deprecated stringify-package@1.0.1: This module is not used anymore, and has been replaced by @npmcli/package-json
npm warn deprecated @babel/plugin-proposal-class-properties@7.18.6: This proposal has been merged to the ECMAScript standard and thus this plugin is no longer maintained. Please use @babel/plugin-transform-class                     -properties instead.
npm warn deprecated apollo-datasource@3.3.2: The `apollo-datasource` package is part of Apollo Server v2 and v3, which are now deprecated (end-of-life October 22nd 2023 and October 22nd 2024, respectively). See https://www.apollographql.com/docs/apollo-server/previous-versions/ for more details.
npm warn deprecated apollo-server-plugin-base@3.7.2: The `apollo-server-plugin-base` package is part of Apollo Server v2 and v3, which are now deprecated (end-of-life October 22nd 2023 and October 22nd 2024, respectively). This package's functionality is now found in the `@apollo/server` package. See https://www.apollographql.com/docs/apollo-server/previous-versions/ for more details.
npm warn deprecated apollo-server-types@3.8.0: The `apollo-server-types` package is part of Apollo Server v2 and v3, which are now deprecated (end-of-life October 22nd 2023 and October 22nd 2024, respectively). This package's functionality is now found in the `@apollo/server` package. See https://www.apollographql.com/docs/apollo-server/previous-versions/ for more details.
npm warn deprecated apollo-server-errors@3.3.1: The `apollo-server-errors` package is part of Apollo Server v2 and v3, which are now deprecated (end-of-life October 22nd 2023 and October 22nd 2024, respectively). This package's functionality is now found in the `@apollo/server` package. See https://www.apollographql.com/docs/apollo-server/previous-versions/ for more details.
npm warn deprecated @babel/plugin-proposal-object-rest-spread@7.20.7: This proposal has been merged to the ECMAScript standard and thus this plugin is no longer maintained. Please use @babel/plugin-transform-object-rest-spread instead.
npm warn deprecated @apollo/server-plugin-landing-page-graphql-playground@4.0.0: The use of GraphQL Playground in Apollo Server was supported in previous versions, but this is no longer the case as of December 31, 2022. This package exists for v4 migration purposes only. We do not intend to resolve security issues or other bugs with this package if they arise, so please migrate away from this to [Apollo Server's default Explorer](https://www.apollographql.com/docs/apollo-server/api/plugin/landing-pages) as soon as possible.
npm warn deprecated apollo-server-env@4.2.1: The `apollo-server-env` package is part of Apollo Server v2 and v3, which are now deprecated (end-of-life October 22nd 2023 and October 22nd 2024, respectively). This package's functionality is now found in the `@apollo/utils.fetcher` package. See https://www.apollographql.com/docs/apollo-server/previous-versions/ for more details.
npm warn deprecated apollo-reporting-protobuf@3.4.0: The `apollo-reporting-protobuf` package is part of Apollo Server v2 and v3, which are now deprecated (end-of-life October 22nd 2023 and October 22nd 2024, respectively). This package's functionality is now found in the `@apollo/usage-reporting-protobuf` package. See https://www.apollographql.com/docs/apollo-server/previous-versions/ for more details.
npm warn deprecated subscriptions-transport-ws@0.11.0: The `subscriptions-transport-ws` package is no longer maintained. We recommend you use `graphql-ws` instead. For help migrating Apollo software to `graphql-ws`, see https://www.apollographql.com/docs/apollo-server/data/subscriptions/#switching-from-subscriptions-transport-ws    For general help using `graphql-ws`, see https://github.com/enisdenjo/graphql-ws/blob/master/README.md

> thunderhub@0.13.31 prepare
> husky install

husky - Git hooks installed

added 1949 packages, and audited 1950 packages in 46s

251 packages are looking for funding
  run `npm fund` for details

23 vulnerabilities (2 low, 6 moderate, 15 high)

To address issues that do not require attention, run:
  npm audit fix

To address all issues (including breaking changes), run:
  npm audit fix --force

Run `npm audit` for details.
npm notice
npm notice New patch version of npm available! 10.8.1 -> 10.8.2
npm notice Changelog: https://github.com/npm/cli/releases/tag/v10.8.2
npm notice To update run: npm install -g npm@10.8.2
npm notice
```

</details>

{% hint style="info" %}
**(Optional)** Improve your privacy by opting out of Next.js [telemetry](https://nextjs.org/telemetry)

```bash
npx next telemetry disable
```

Expected output:

```
Your preference has been saved to /home/thunderhub/.config/nextjs-nodejs/config.json.

Status: Disabled

You have opted-out of Next.js' anonymous telemetry program.
No data will be collected from your machine.
Learn more: https://nextjs.org/telemetry
```
{% endhint %}

* Build it

```sh
npm run build
```

<details>

<summary><strong>Example</strong> of expected output ⬇️</summary>

```
> thunderhub@0.13.31 prebuild
> rimraf dist && rimraf .next


> thunderhub@0.13.31 build
> npm run build:nest && npm run build:next


> thunderhub@0.13.31 build:nest
> nest build


> thunderhub@0.13.31 build:next
> cd src/client && next build


./src/components/chart/BarChart.tsx
61:6  Warning: React Hook useMemo has a missing dependency: 'dataKey'. Either include it or remove the dependency array.  react-hooks/exhaustive-deps

./src/components/chart/HorizontalBarChart.tsx
139:6  Warning: React Hook useMemo has a missing dependency: 'maxValue'. Either include it or remove the dependency array.  react-hooks/exhaustive-deps

./src/components/table/DebouncedInput.tsx
30:6  Warning: React Hook useEffect has missing dependencies: 'debounce' and 'onChange'. Either include them or remove the dependency array. If 'onChange' changes too often, find the parent component that defines it and wrap that definition in useCallback.  react-hooks/exhaustive-deps

info  - Need to disable some ESLint rules? Learn more here: https://nextjs.org/docs/basic-features/eslint#disabling-rules
 ✓ Linting and checking validity of types
Browserslist: caniuse-lite is outdated. Please run:
  npx browserslist@latest --update-db
  Why you should do it regularly: https://github.com/browserslist/browserslist#browsers-data-updating
 ✓ Creating an optimized production build
 ✓ Compiled successfully
 ✓ Collecting page data
 ✓ Collecting build traces
 ✓ Finalizing page optimization

Route (pages)                              Size     First Load JS
┌ λ /                                      23.9 kB         557 kB
├   /_app                                  0 B             243 kB
├ λ /404                                   344 B           243 kB
├ λ /amboss                                3.92 kB         250 kB
├ λ /chain                                 5.69 kB         265 kB
├ λ /channels                              6.61 kB         310 kB
├ λ /channels/[slug]                       4.44 kB         250 kB
├ λ /chat                                  6.63 kB         255 kB
├ λ /dashboard                             586 B           247 kB
├ λ /forwards                              23.5 kB         545 kB
├ λ /leaderboard                           3.62 kB         281 kB
├ λ /lnmarkets                             5.2 kB          248 kB
├ λ /login                                 5.54 kB         249 kB
├ λ /peers                                 6.29 kB         265 kB
├ λ /rebalance                             9.28 kB         287 kB
├ λ /settings                              8.66 kB         257 kB
├ λ /settings/dashboard                    458 B           247 kB
├ λ /sso                                   2.78 kB         246 kB
├ λ /stats                                 7.02 kB         253 kB
├ λ /swap                                  11.2 kB         289 kB
├ λ /tools                                 7.38 kB         250 kB
└ λ /transactions                          5.08 kB         523 kB
+ First Load JS shared by all              247 kB
  ├ chunks/framework-80ea8c0f440c6a32.js   45.4 kB
  ├ chunks/main-5aa2e2aecccdc7ca.js        33 kB
  ├ chunks/pages/_app-43ed1c524f6479ab.js  162 kB
  ├ chunks/webpack-bafa1815dd7342f2.js     2.17 kB
  └ css/9f506b76c3634369.css               4.22 kB

λ  (Server)  server-side renders at runtime (uses getInitialProps or getServerSideProps)
```

</details>

{% hint style="info" %}
This process can take quite **a long time**, 10-15 minutes or more, depending on the performance of your device. Please be patient until the prompt shows again
{% endhint %}

* Check the correct installation by requesting the version

```bash
head -n 3 /home/thunderhub/thunderhub/package.json | grep version
```

**Example** of expected output:

```
"version": "0.13.19",
```

## Configuration

* Copy the configuration file template

```sh
cp .env .env.local
```

* Edit the configuration file

```sh
nano .env.local
```

* Uncomment and edit the following line to match the next. Save and exit

```
ACCOUNT_CONFIG_PATH='/home/thunderhub/thunderhub/thubConfig.yaml'
```

* Create a new`thubConfig.yaml` file

```sh
nano thubConfig.yaml
```

* Copy and paste the following information

<pre class="language-yaml"><code class="lang-yaml">masterPassword: '<a data-footnote-ref href="#user-content-fn-1">PASSWORD</a>'
accounts:
  - name: 'RaMiX'
    serverUrl: '127.0.0.1:10009'
    macaroonPath: '/data/lnd/data/chain/bitcoin/mainnet/admin.macaroon'
    certificatePath: '/data/lnd/tls.cert'
    password: '[E] ThunderHub password'
</code></pre>

{% hint style="info" %}
Replace the **`[E] ThunderHub password`** to your one, keeping quotes \[' ']
{% endhint %}

* **(Optional)** You can pre-enable automatic healthchecks ping, and/or channel backups to Amboss before starting ThunderHub by adding some lines **at the end of the file** (**without indentation**)

Enable auto-backups:

```
backupsEnabled: true
```

Enable-auto healthchecks:

```
healthCheckPingEnabled: true
```

{% hint style="info" %}
> Anyway is possible to enable this later using the ThunderHub interface that will be explained in the [Enable auto backups and healthcheck notifications](web-app.md#enable-auto-backups-and-healthcheck-notifications-to-the-amboss-account) extra section

> Keep in mind that if you stop ThunderHub, Amboss will interpret that your node is offline because the connection is established between ThunderHub <> Amboss to send healthchecks pings
{% endhint %}

{% hint style="info" %}
These features are not available for a testnet node
{% endhint %}

* Exit `thunderhub` user session to return to the `admin` user session

```sh
exit
```

### Create systemd service

* As user `admin`, create the service file

```sh
sudo nano /etc/systemd/system/thunderhub.service
```

* Paste the following configuration. Save and exit

<pre><code># RaMiX: systemd unit for Thunderhub
# /etc/systemd/system/thunderhub.service

[Unit]
Description=ThunderHub
<strong>Requires=lnd.service
</strong>After=lnd.service

[Service]
WorkingDirectory=/home/thunderhub/thunderhub
ExecStart=/usr/bin/npm run start

User=thunderhub
Group=thunderhub

# Process management
####################
TimeoutSec=300

# Hardening Measures
####################
PrivateTmp=true
ProtectSystem=full
NoNewPrivileges=true
PrivateDevices=true

[Install]
WantedBy=multi-user.target
</code></pre>

* Enable autoboot **(optional)**

```sh
sudo systemctl enable thunderhub
```

* Prepare "thunderhub" monitoring by the systemd journal and check the log output. You can exit monitoring at any time with `Ctrl-C`

```bash
journalctl -fu thunderhub
```

## Run

To keep an eye on the software movements, [start your SSH program](../index-1/remote-access.md#access-with-secure-shell) straight forward (eg. PuTTY) a second time, connect to the RaMiX node, and log in as "admin"

* Start the service

```sh
sudo systemctl start thunderhub
```

<details>

<summary><strong>Example</strong> of expected output on the first terminal with <code>journalctl -fu thunderhub</code> ⬇️</summary>

```
Feb 27 09:32:54 ramix systemd[1]: Started thunderhub.service - ThunderHub.
Feb 27 09:32:54 ramix npm[1551967]: > thunderhub@0.15.1 start
Feb 27 09:32:54 ramix npm[1551967]: > cross-env NODE_ENV=production nest start
Feb 27 09:33:17 ramix npm[1552018]: [Nest] 1552018  - 27/02/2026, 09:33:17     LOG [NestFactory] Starting Nest application...
Feb 27 09:33:18 ramix npm[1552018]: Getting production env variables.
Feb 27 09:33:18 ramix npm[1552018]: [Nest] 1552018  - 27/02/2026, 09:33:18     LOG [InstanceLoader] AppModule dependencies initialized +133ms
Feb 27 09:33:18 ramix npm[1552018]: [Nest] 1552018  - 27/02/2026, 09:33:18     LOG [InstanceLoader] PassportModule dependencies initialized +1ms
Feb 27 09:33:18 ramix npm[1552018]: [Nest] 1552018  - 27/02/2026, 09:33:18     LOG [InstanceLoader] LndModule dependencies initialized +0ms
Feb 27 09:33:18 ramix npm[1552018]: [Nest] 1552018  - 27/02/2026, 09:33:18     LOG [InstanceLoader] ApiModule dependencies initialized +1ms
Feb 27 09:33:18 ramix npm[1552018]: [Nest] 1552018  - 27/02/2026, 09:33:18     LOG [InstanceLoader] MainModule dependencies initialized +0ms
Feb 27 09:33:18 ramix npm[1552018]: [Nest] 1552018  - 27/02/2026, 09:33:18     LOG [InstanceLoader] ConfigHostModule dependencies initialized +4ms
Feb 27 09:33:18 ramix npm[1552018]: [Nest] 1552018  - 27/02/2026, 09:33:18     LOG [InstanceLoader] DiscoveryModule dependencies initialized +0ms
Feb 27 09:33:18 ramix npm[1552018]: [Nest] 1552018  - 27/02/2026, 09:33:18     LOG [InstanceLoader] ConfigModule dependencies initialized +12ms
Feb 27 09:33:18 ramix npm[1552018]: [Nest] 1552018  - 27/02/2026, 09:33:18     LOG [InstanceLoader] ConfigModule dependencies initialized +1ms
Feb 27 09:33:18 ramix npm[1552018]: [Nest] 1552018  - 27/02/2026, 09:33:18     LOG [InstanceLoader] ScheduleModule dependencies initialized +0ms
Feb 27 09:33:18 ramix npm[1552018]: [Nest] 1552018  - 27/02/2026, 09:33:18     LOG [InstanceLoader] ClientConfigModule dependencies initialized +0ms
Feb 27 09:33:18 ramix npm[1552018]: [Nest] 1552018  - 27/02/2026, 09:33:18     LOG [InstanceLoader] ThrottlerModule dependencies initialized +9ms
Feb 27 09:33:18 ramix npm[1552018]: [Nest] 1552018  - 27/02/2026, 09:33:18     LOG [InstanceLoader] JwtModule dependencies initialized +0ms
Feb 27 09:33:18 ramix npm[1552018]: [Nest] 1552018  - 27/02/2026, 09:33:18     LOG [InstanceLoader] ServeStaticModule dependencies initialized +0ms
Feb 27 09:33:18 ramix npm[1552018]: [Nest] 1552018  - 27/02/2026, 09:33:18     LOG [InstanceLoader] WinstonModule dependencies initialized +2ms
Feb 27 09:33:18 ramix npm[1552018]: [Nest] 1552018  - 27/02/2026, 09:33:18     LOG [InstanceLoader] FilesModule dependencies initialized +0ms
Feb 27 09:33:18 ramix npm[1552018]: [Nest] 1552018  - 27/02/2026, 09:33:18     LOG [InstanceLoader] FetchModule dependencies initialized +1ms
Feb 27 09:33:18 ramix npm[1552018]: [Nest] 1552018  - 27/02/2026, 09:33:18     LOG [InstanceLoader] AuthenticationModule dependencies initialized +0ms
Feb 27 09:33:18 ramix npm[1552018]: [Nest] 1552018  - 27/02/2026, 09:33:18     LOG [InstanceLoader] GraphQLSchemaBuilderModule dependencies initialized +0ms
Feb 27 09:33:18 ramix npm[1552018]: [Nest] 1552018  - 27/02/2026, 09:33:18     LOG [InstanceLoader] AccountsModule dependencies initialized +2ms
Feb 27 09:33:18 ramix npm[1552018]: [Nest] 1552018  - 27/02/2026, 09:33:18     LOG [InstanceLoader] MempoolModule dependencies initialized +0ms
Feb 27 09:33:18 ramix npm[1552018]: [Nest] 1552018  - 27/02/2026, 09:33:18     LOG [InstanceLoader] BlockstreamModule dependencies initialized +0ms
Feb 27 09:33:18 ramix npm[1552018]: [Nest] 1552018  - 27/02/2026, 09:33:18     LOG [InstanceLoader] BitcoinModule dependencies initialized +0ms
Feb 27 09:33:18 ramix npm[1552018]: [Nest] 1552018  - 27/02/2026, 09:33:18     LOG [InstanceLoader] GithubModule dependencies initialized +0ms
Feb 27 09:33:18 ramix npm[1552018]: [Nest] 1552018  - 27/02/2026, 09:33:18     LOG [InstanceLoader] UserConfigModule dependencies initialized +5ms
Feb 27 09:33:18 ramix npm[1552018]: [Nest] 1552018  - 27/02/2026, 09:33:18     LOG [InstanceLoader] NodeModule dependencies initialized +0ms
Feb 27 09:33:18 ramix npm[1552018]: [Nest] 1552018  - 27/02/2026, 09:33:18     LOG [InstanceLoader] AccountModule dependencies initialized +0ms
Feb 27 09:33:18 ramix npm[1552018]: [Nest] 1552018  - 27/02/2026, 09:33:18     LOG [InstanceLoader] AuthenticationModule dependencies initialized +0ms
Feb 27 09:33:18 ramix npm[1552018]: [Nest] 1552018  - 27/02/2026, 09:33:18     LOG [InstanceLoader] SseModule dependencies initialized +1ms
Feb 27 09:33:18 ramix npm[1552018]: [Nest] 1552018  - 27/02/2026, 09:33:18     LOG [InstanceLoader] WalletModule dependencies initialized +1ms
Feb 27 09:33:18 ramix npm[1552018]: [Nest] 1552018  - 27/02/2026, 09:33:18     LOG [InstanceLoader] NetworkModule dependencies initialized +1ms
Feb 27 09:33:18 ramix npm[1552018]: [Nest] 1552018  - 27/02/2026, 09:33:18     LOG [InstanceLoader] EdgeModule dependencies initialized +0ms
Feb 27 09:33:18 ramix npm[1552018]: [Nest] 1552018  - 27/02/2026, 09:33:18     LOG [InstanceLoader] TransactionsModule dependencies initialized +0ms
Feb 27 09:33:18 ramix npm[1552018]: [Nest] 1552018  - 27/02/2026, 09:33:18     LOG [InstanceLoader] ToolsModule dependencies initialized +0ms
Feb 27 09:33:18 ramix npm[1552018]: [Nest] 1552018  - 27/02/2026, 09:33:18     LOG [InstanceLoader] MacaroonModule dependencies initialized +0ms
Feb 27 09:33:18 ramix npm[1552018]: [Nest] 1552018  - 27/02/2026, 09:33:18     LOG [InstanceLoader] PeerModule dependencies initialized +0ms
Feb 27 09:33:18 ramix npm[1552018]: [Nest] 1552018  - 27/02/2026, 09:33:18     LOG [InstanceLoader] ChainModule dependencies initialized +0ms
Feb 27 09:33:18 ramix npm[1552018]: [Nest] 1552018  - 27/02/2026, 09:33:18     LOG [InstanceLoader] ForwardsModule dependencies initialized +0ms
Feb 27 09:33:18 ramix npm[1552018]: [Nest] 1552018  - 27/02/2026, 09:33:18     LOG [InstanceLoader] HealthModule dependencies initialized +1ms
Feb 27 09:33:18 ramix npm[1552018]: [Nest] 1552018  - 27/02/2026, 09:33:18     LOG [InstanceLoader] InvoicesModule dependencies initialized +0ms
Feb 27 09:33:18 ramix npm[1552018]: [Nest] 1552018  - 27/02/2026, 09:33:18     LOG [InstanceLoader] ChatModule dependencies initialized +0ms
Feb 27 09:33:18 ramix npm[1552018]: [Nest] 1552018  - 27/02/2026, 09:33:18     LOG [InstanceLoader] NodeModule dependencies initialized +0ms
Feb 27 09:33:18 ramix npm[1552018]: [Nest] 1552018  - 27/02/2026, 09:33:18     LOG [InstanceLoader] ChannelsModule dependencies initialized +0ms
Feb 27 09:33:18 ramix npm[1552018]: [Nest] 1552018  - 27/02/2026, 09:33:18     LOG [InstanceLoader] AuthModule dependencies initialized +0ms
Feb 27 09:33:18 ramix npm[1552018]: [Nest] 1552018  - 27/02/2026, 09:33:18     LOG [InstanceLoader] BoltzModule dependencies initialized +0ms
Feb 27 09:33:18 ramix npm[1552018]: [Nest] 1552018  - 27/02/2026, 09:33:18     LOG [InstanceLoader] LnUrlModule dependencies initialized +2ms
Feb 27 09:33:18 ramix npm[1552018]: [Nest] 1552018  - 27/02/2026, 09:33:18     LOG [InstanceLoader] DataloaderModule dependencies initialized +0ms
Feb 27 09:33:18 ramix npm[1552018]: [Nest] 1552018  - 27/02/2026, 09:33:18     LOG [InstanceLoader] AmbossModule dependencies initialized +0ms
Feb 27 09:33:18 ramix npm[1552018]: [Nest] 1552018  - 27/02/2026, 09:33:18     LOG [InstanceLoader] SubModule dependencies initialized +0ms
Feb 27 09:33:18 ramix npm[1552018]: [Nest] 1552018  - 27/02/2026, 09:33:18     LOG [InstanceLoader] GraphQLModule dependencies initialized +4ms
Feb 27 09:33:18 ramix npm[1552018]: {
Feb 27 09:33:18 ramix npm[1552018]:   context: 'RoutesResolver',
Feb 27 09:33:18 ramix npm[1552018]:   level: 'info',
Feb 27 09:33:18 ramix npm[1552018]:   message: 'SseController {/api/sse}:',
Feb 27 09:33:18 ramix npm[1552018]:   timestamp: '2026-02-27T08:33:18.126Z'
Feb 27 09:33:18 ramix npm[1552018]: }
Feb 27 09:33:18 ramix npm[1552018]: {
Feb 27 09:33:18 ramix npm[1552018]:   context: 'RouterExplorer',
Feb 27 09:33:18 ramix npm[1552018]:   level: 'info',
Feb 27 09:33:18 ramix npm[1552018]:   message: 'Mapped {/api/sse/events, GET} route',
Feb 27 09:33:18 ramix npm[1552018]:   timestamp: '2026-02-27T08:33:18.140Z'
Feb 27 09:33:18 ramix npm[1552018]: }
Feb 27 09:33:18 ramix npm[1552018]: {
Feb 27 09:33:18 ramix npm[1552018]:   context: 'RoutesResolver',
Feb 27 09:33:18 ramix npm[1552018]:   level: 'info',
Feb 27 09:33:18 ramix npm[1552018]:   message: 'ClientConfigController {/api}:',
Feb 27 09:33:18 ramix npm[1552018]:   timestamp: '2026-02-27T08:33:18.140Z'
Feb 27 09:33:18 ramix npm[1552018]: }
Feb 27 09:33:18 ramix npm[1552018]: {
Feb 27 09:33:18 ramix npm[1552018]:   context: 'RouterExplorer',
Feb 27 09:33:18 ramix npm[1552018]:   level: 'info',
Feb 27 09:33:18 ramix npm[1552018]:   message: 'Mapped {/api/config, GET} route',
Feb 27 09:33:18 ramix npm[1552018]:   timestamp: '2026-02-27T08:33:18.143Z'
Feb 27 09:33:18 ramix npm[1552018]: }
Feb 27 09:33:18 ramix npm[1552018]: {
Feb 27 09:33:18 ramix npm[1552018]:   message: 'Server accounts that will be available: ramix',
Feb 27 09:33:18 ramix npm[1552018]:   level: 'info',
Feb 27 09:33:18 ramix npm[1552018]:   timestamp: '2026-02-27T08:33:18.186Z'
Feb 27 09:33:18 ramix npm[1552018]: }
Feb 27 09:33:19 ramix npm[1552018]: {
Feb 27 09:33:19 ramix npm[1552018]:   context: 'GraphQLModule',
Feb 27 09:33:19 ramix npm[1552018]:   level: 'info',
Feb 27 09:33:19 ramix npm[1552018]:   message: 'Mapped {/graphql, POST} route',
Feb 27 09:33:19 ramix npm[1552018]:   timestamp: '2026-02-27T08:33:19.962Z'
Feb 27 09:33:19 ramix npm[1552018]: }
Feb 27 09:33:19 ramix npm[1552018]: {
Feb 27 09:33:19 ramix npm[1552018]:   context: 'NestApplication',
Feb 27 09:33:19 ramix npm[1552018]:   level: 'info',
Feb 27 09:33:19 ramix npm[1552018]:   message: 'Nest application successfully started',
Feb 27 09:33:19 ramix npm[1552018]:   timestamp: '2026-02-27T08:33:19.996Z'
Feb 27 09:33:19 ramix npm[1552018]: }
Feb 27 09:33:20 ramix npm[1552018]: Application is running on: http://[::1]:3001
Feb 27 09:33:20 ramix npm[1552018]: (node:1552018) [DEP0123] DeprecationWarning: Setting the TLS ServerName to an IP address is not permitted by RFC 6066. This will be ignored in a future version.
Feb 27 09:33:20 ramix npm[1552018]: (Use `node --trace-deprecation ...` to show where the warning was created)
Feb 27 09:33:20 ramix npm[1552018]: {
Feb 27 09:33:20 ramix npm[1552018]:   message: 'Connected to 2FakTorLNtest4🧪🧬(030f289f0f)[btctestnet4]',
Feb 27 09:33:20 ramix npm[1552018]:   level: 'info',
Feb 27 09:33:20 ramix npm[1552018]:   timestamp: '2026-02-27T08:33:20.295Z'
Feb 27 09:33:20 ramix npm[1552018]: }
Feb 27 09:33:20 ramix npm[1552018]: {
Feb 27 09:33:20 ramix npm[1552018]:   connections: '2FakTorLNtest4🧪🧬(030f289f0f)[btctestnet4]',
Feb 27 09:33:20 ramix npm[1552018]:   level: 'info',
Feb 27 09:33:20 ramix npm[1552018]:   message: 'Invoice subscription',
Feb 27 09:33:20 ramix npm[1552018]:   timestamp: '2026-02-27T08:33:20.298Z'
Feb 27 09:33:20 ramix npm[1552018]: }
Feb 27 09:33:20 ramix npm[1552018]: {
Feb 27 09:33:20 ramix npm[1552018]:   connections: '2FakTorLNtest4🧪🧬(030f289f0f)[btctestnet4]',
Feb 27 09:33:20 ramix npm[1552018]:   level: 'info',
Feb 27 09:33:20 ramix npm[1552018]:   message: 'Payment subscription',
Feb 27 09:33:20 ramix npm[1552018]:   timestamp: '2026-02-27T08:33:20.304Z'
Feb 27 09:33:20 ramix npm[1552018]: }
Feb 27 09:33:20 ramix npm[1552018]: {
Feb 27 09:33:20 ramix npm[1552018]:   connections: '2FakTorLNtest4🧪🧬(030f289f0f)[btctestnet4]',
Feb 27 09:33:20 ramix npm[1552018]:   level: 'info',
Feb 27 09:33:20 ramix npm[1552018]:   message: 'Forward subscription',
Feb 27 09:33:20 ramix npm[1552018]:   timestamp: '2026-02-27T08:33:20.309Z'
Feb 27 09:33:20 ramix npm[1552018]: }
Feb 27 09:33:20 ramix npm[1552018]: {
Feb 27 09:33:20 ramix npm[1552018]:   connections: '2FakTorLNtest4🧪🧬(030f289f0f)[btctestnet4]',
Feb 27 09:33:20 ramix npm[1552018]:   level: 'info',
Feb 27 09:33:20 ramix npm[1552018]:   message: 'Channels subscription',
Feb 27 09:33:20 ramix npm[1552018]:   timestamp: '2026-02-27T08:33:20.311Z'
Feb 27 09:33:20 ramix npm[1552018]: }
Feb 27 09:33:20 ramix npm[1552018]: {
Feb 27 09:33:20 ramix npm[1552018]:   connections: '2FakTorLNtest4🧪🧬(030f289f0f)[btctestnet4]',
Feb 27 09:33:20 ramix npm[1552018]:   level: 'info',
Feb 27 09:33:20 ramix npm[1552018]:   message: 'Backup subscription',
Feb 27 09:33:20 ramix npm[1552018]:   timestamp: '2026-02-27T08:33:20.315Z'
Feb 27 09:33:20 ramix npm[1552018]: }

```

</details>

### Validation

* Ensure the service is working and listening on the default `3001` port and the HTTPS `4002` port

```bash
sudo ss -tulpn | grep -v 'dotnet' | grep -E '(:4002|:3001)'
```

Expected output:

```
tcp   LISTEN 0      511          0.0.0.0:4002       0.0.0.0:*    users:(("nginx",pid=541834,fd=6),("nginx",pid=541833,fd=6),("nginx",pid=541832,fd=6),("nginx",pid=541831,fd=6),("nginx",pid=541041,fd=6))
tcp   LISTEN 0      511                *:3001             *:*    users:(("MainThread",pid=1550842,fd=21))
```

{% hint style="info" %}
> Your browser will display a warning because we use a self-signed SSL certificate. We can do nothing about that because we would need a proper domain name (e.g., https://yournode.com) to get an official certificate that browsers recognize. Click on "Advanced" and proceed to the ThunderHub web interface

> Now point your browser to `https://ramix.local:4002` or the IP address (e.g. `https://192.168.x.xxx:4002`). You should see the home page of ThunderHub
{% endhint %}

{% hint style="success" %}
Congrats! You now have ThunderHub up and running
{% endhint %}

## Extras (optional)

### Remote access over Tor

* With the user `admin`, edit the `torrc` file

```sh
sudo nano +63 /etc/tor/torrc --linenumbers
```

* Add the following lines in the "location hidden services" section, below "`## This section is just for location-hidden services ##`". Save and exit

```
# Hidden Service ThunderHub
HiddenServiceDir /var/lib/tor/hidden_service_thunderhub/
HiddenServiceEnableIntroDoSDefense 1
HiddenServicePoWDefensesEnabled 1
HiddenServicePort 80 127.0.0.1:3001
```

* Reload Tor to apply changes

```sh
sudo systemctl reload tor
```

* Get your Onion address

```sh
sudo cat /var/lib/tor/hidden_service_thunderhub/hostname
```

Expected output:

```
abcdefg..............xyz.onion
```

* With the [Tor browser](https://www.torproject.org), you can access this onion address from any device

### Access to your Amboss node account

* In the "**Home**" screen - "**Quick Actions**" section, click on the Amboss icon "**Login**", wait for the top right corner notification to show you "**Logged in**", and click again on the Amboss icon "**Go to**". This will open a secondary tab in your browser to access your Amboss account node

{% hint style="warning" %}
If you can't do "**Login**", maybe the cause is that you don't have a **public** channel opened yet. **You'll need at least one public channel that has been open for a few days.** Planning to open a small-sized public channel to be connected with some Lightning Network peers or directly to the [Amboss node](https://amboss.space/es/node/03006fcf3312dae8d068ea297f58e2bd00ec1ffe214b793eda46966b6294a53ce6). More info on [Amboss docs](https://amboss.tech/docs)
{% endhint %}

* Making sure we are connected to the [Amboss account](https://amboss.space/settings?page=account), now back to Thunderhub for the next steps

### Enable auto backups and healthcheck notifications to the Amboss account

#### Enable automatic backups to Amboss

1. In ThunderHub, from the left sidebar, click on 🌍**Amboss.**
2. In the **Backups section**, push the **Push** button to test and push the first backup to Amboss. If all is good, you could enable automatic backups to Amboss by pushing on **Enable** just above; now the backup file encrypted will be updated automatically on Amboss for every channel opening and closing.
3. Go to the Amboss website -> [backups section](https://amboss.space/settings?page=backups).
4. Ensure that the last date of the backup is the same as before.

<figure><img src="../.gitbook/assets/pushed-backup-amboss.png" alt="" width="563"><figcaption></figcaption></figure>

{% hint style="info" %}
> You could test that the possible recovery process would be available, by clicking on the "**Get**" button and copying the entire string, then going back to the Thunderhub from the left sidebar, clicking on "**Tools",** going to the "Backups" section -> "Verify Channels Backup" -> click on "**Verify"** button, paste the before string copied and click on "Verify" button again. A green banner "**Valid backup String**" should appear.

> Also is recommended to download the backup file from ThunderHub and store locally it in a safe place for future recovery. You can do this "**Tools**" section in Thunderhub, "**Backups**" -> "Backup all channels" -> click the "**Download**" button.
{% endhint %}

#### Enable automatic healthcheck pings to Amboss

1. In ThunderHub, from the left sidebar, click on 🌍**Amboss.**
2. Go to the **Healthchecks section** and push the "**Enable**" button to enable automatic healthcheck pings to Amboss.
3. Now go to the Amboss [Monitoring section](https://amboss.space/settings?page=monitoring), and configure "Healthcheck Settings" as you wish.
4. Go to the [Notifications section](https://amboss.space/settings?page=notifications) to enable the different notification methods that you wish to be notified.

{% hint style="info" %}
> Feel free to link to the Telegram bot notifications, enable different notifications, complete your public node profile in Amboss, and other things in the different sections of your account

> Keep in mind that if you stop ThunderHub, Amboss will interpret that your node is offline because the connection is established between ThunderHub <-> Ambos to send healthchecks pings
{% endhint %}

### Recovering channels using the ThunderHub method

After possible data corruption of your LND node, ensure that this old node is completely off before starting the recovery.

Once you have synced the new node, on-chain recovered with seeds, full on-chain re-scan complete, and Thunderhub installed and running, go to the Thunderhub dashboard.

1. From the left sidebar, click on "**Tools"**, and go to the "Backups" section -> "**Recover Funds from Channels**" -> push the "**Recover**" button.
2. In this box, enter the complete string text that contains your manually downloaded channels backup file in the step before, or use the string using the content of the latest Amboss automatic backup (recommended), and push the " Recover " button again.

{% hint style="info" %}
All of the channels that you had opened in your old node will be forced closed, and they will appear in the "Pending" tab in the "Channels" section until closings are confirmed. Check the logs of LND to see how the recovery process is executed and get more information about it
{% endhint %}

{% hint style="danger" %}
Use this guide as a last resort if you have lost access to your node or are unable to start LND due to a fatal error. This guide will close all your channels. Your funds will become available on-chain at varying speeds
{% endhint %}

## Upgrade

Updating to a [new release](https://github.com/apotdevin/thunderhub/releases) should be straightforward.

* Stay logged in with the user `admin`, stop the service

```sh
sudo systemctl stop thunderhub
```

* Change to the `thunderhub` user

```sh
sudo su - thunderhub
```

* Go to the thunderhub folder

```sh
cd thunderhub
```

* Set the environment variable version

```bash
VERSION=0.15.1
```

* Pull the changes from GitHub

```bash
git pull https://github.com/apotdevin/thunderhub.git v$VERSION
```

<details>

<summary><strong>Example</strong> of expected output ⬇️</summary>

```
remote: Enumerating objects: 914, done.
remote: Counting objects: 100% (560/560), done.
remote: Compressing objects: 100% (279/279), done.
remote: Total 914 (delta 263), reused 370 (delta 236), pack-reused 354 (from 3)
Receiving objects: 100% (914/914), 885.90 KiB | 2.34 MiB/s, done.
Resolving deltas: 100% (376/376), completed with 95 local objects.
From https://github.com/apotdevin/thunderhub
 * tag                 v0.15.1    -> FETCH_HEAD
Updating b2ca712e..d535ead1
Fast-forward
 .dockerignore                                                                      |     7 +-
 .eslintignore                                                                      |     6 -
 .eslintrc.js                                                                       |    26 -
 .github/workflows/docker-release.yml                                               |    55 +
 .gitignore                                                                         |     6 +-
 .husky/pre-commit                                                                  |     3 -
 .nvmrc                                                                             |     2 +-
 CHANGELOG.md                                                                       |    21 +
 Dockerfile                                                                         |    35 +-
 [...]
```

</details>

* Install all the necessary modules

```bash
npm install
```

<details>

<summary><strong>Example</strong> of expected output ⬇️</summary>

```
npm warn ERESOLVE overriding peer dependency
npm warn While resolving: @apollo/server-plugin-landing-page-graphql-playground@4.0.1
npm warn Found: @apollo/server@5.4.0
npm warn node_modules/@apollo/server
npm warn   @apollo/server@"^5.4.0" from the root project
npm warn   2 more (@as-integrations/express5, @nestjs/apollo)
npm warn
npm warn Could not resolve dependency:
npm warn peer @apollo/server@"^4.0.0" from @apollo/server-plugin-landing-page-graphql-playground@4.0.1
npm warn node_modules/@apollo/server-plugin-landing-page-graphql-playground
npm warn   @apollo/server-plugin-landing-page-graphql-playground@"4.0.1" from @nestjs/apollo@13.2.4
npm warn   node_modules/@nestjs/apollo
npm warn
npm warn Conflicting peer dependency: @apollo/server@4.13.0
npm warn node_modules/@apollo/server
npm warn   peer @apollo/server@"^4.0.0" from @apollo/server-plugin-landing-page-graphql-playground@4.0.1
npm warn   node_modules/@apollo/server-plugin-landing-page-graphql-playground
npm warn     @apollo/server-plugin-landing-page-graphql-playground@"4.0.1" from @nestjs/apollo@13.2.4
npm warn     node_modules/@nestjs/apollo
npm warn deprecated @apollo/server-plugin-landing-page-graphql-playground@4.0.1: The use of GraphQL Playground in Apollo Server was supported in previous versions, but this is no longer the case as of December 3 exists for v4 migration purposes only. We do not intend to resolve security issues or other bugs with this package if they arise, so please migrate away from this to [Apollo Server's default Explorer](https://wdocs/apollo-server/api/plugin/landing-pages) as soon as possible.
npm warn deprecated glob@10.5.0: Old versions of glob are not supported, and contain widely publicized security vulnerabilities, which have been fixed in the current version. Please update. Support for old versi(at exorbitant rates) by contacting i@izs.me
npm warn deprecated glob@10.5.0: Old versions of glob are not supported, and contain widely publicized security vulnerabilities, which have been fixed in the current version. Please update. Support for old versi(at exorbitant rates) by contacting i@izs.me
npm warn deprecated glob@10.5.0: Old versions of glob are not supported, and contain widely publicized security vulnerabilities, which have been fixed in the current version. Please update. Support for old versi(at exorbitant rates) by contacting i@izs.me

> thunderhub@0.15.1 prepare
> husky


added 369 packages, removed 832 packages, changed 375 packages, and audited 1545 packages in 3m

224 packages are looking for funding
  run `npm fund` for details

17 vulnerabilities (3 low, 11 moderate, 2 high, 1 critical)

To address issues that do not require attention, run:
  npm audit fix

To address all issues (including breaking changes), run:
  npm audit fix --force

Run `npm audit` for details.
```

</details>

* Build it

<pre class="language-bash"><code class="lang-bash"><strong>npm run build
</strong></code></pre>

<details>

<summary><strong>Example</strong> of expected output ⬇️</summary>

```

> thunderhub@0.15.1 prebuild
> rimraf dist && rimraf src/client/dist


> thunderhub@0.15.1 build
> npm run build:nest && npm run build:client


> thunderhub@0.15.1 build:nest
> nest build


> thunderhub@0.15.1 build:client
> cd src/client && npx vite build

vite v7.3.1 building client environment for production...
✓ 3432 modules transformed.
dist/index.html                                    0.60 kB │ gzip:   0.36 kB
dist/assets/index-C2Q4IGPT.css                    32.24 kB │ gzip:   6.62 kB
dist/assets/SettingsDashboardPage-NuTzbgRs.js      1.59 kB │ gzip:   0.84 kB
dist/assets/DashboardPage-D8gCYArr.js             61.63 kB │ gzip:  20.56 kB
dist/assets/index-CbTrqCDL.js                  1,671.44 kB │ gzip: 533.46 kB

(!) Some chunks are larger than 500 kB after minification. Consider:
- Using dynamic import() to code-split the application
- Use build.rollupOptions.output.manualChunks to improve chunking: https://rollupjs.org/configuration-options/#output-manualchunks
- Adjust chunk size limit for this warning via build.chunkSizeWarningLimit.
```

</details>

* Check the correct update

```bash
head -n 3 /home/thunderhub/thunderhub/package.json | grep version
```

**Example** of expected output:

```
"version": "0.13.20",
```

* Exit to go back to the `admin` user

```bash
exit
```

* Start the service again

```sh
sudo systemctl start thunderhub
```

{% hint style="warning" %}
If the update fails, you probably will have to stop ThunderHub, follow the [Uninstall ThunderHub section](web-app.md#uninstall-thunderhub) to delete `thunderhub` user, and repeat the installation process starting from the [Preparation section](web-app.md#preparation)
{% endhint %}

## Uninstall

### Uninstall service

* With user `admin` , stop thunderhub service

```sh
sudo systemctl stop thunderhub
```

* Disable autoboot (if enabled)

```sh
sudo systemctl disable thunderhub
```

* Delete the service

```sh
sudo rm /etc/systemd/system/thunderhub.service
```

### Delete user & group

* Delete the "thunderhub" user. Do not worry about the `userdel: thunderhub mail spool (/var/mail/thunderhub) not found`

```sh
sudo userdel -rf thunderhub
```

### Uninstall Tor hidden service

* Comment or remove the ThunderHub hidden service lines in torrc. Save and exit

```sh
sudo nano +63 /etc/tor/torrc --linenumbers
```

```
# Hidden Service ThunderHub
#HiddenServiceDir /var/lib/tor/hidden_service_thunderhub/
#HiddenServiceEnableIntroDoSDefense 1
#HiddenServicePoWDefensesEnabled 1
#HiddenServicePort 80 127.0.0.1:3001
```

* Reload the Tor config to apply changes

```sh
sudo systemctl reload tor
```

### Uninstall reverse proxy & FW configuration

* Ensure you are logged in as the user `admin`, delete the reverse proxy config file

```bash
sudo rm /etc/nginx/sites-available/thunderhub-reverse-proxy.conf
```

* Delete the symbolic link

```bash
sudo rm /etc/nginx/sites-enabled/thunderhub-reverse-proxy.conf
```

* Test Nginx configuration

```bash
sudo nginx -t
```

Expected output:

```
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

* Reload the Nginx configuration to apply changes

```bash
sudo systemctl reload nginx
```

* Display the UFW firewall rules and note the numbers of the rules for ThunderHub (e.g, "X" below)

```sh
sudo ufw status numbered
```

Expected output:

```
[X] 4002    ALLOW IN    Anywhere         # allow ThunderHub SSL from anywhere
```

* Delete related ThunderHub rules (check that the rule to be deleted is the correct one and type "y" and "Enter" when prompted)

```sh
sudo ufw delete X
```

## Port reference

<table><thead><tr><th align="center">Port</th><th width="100">Protocol<select><option value="ipZ6Euow0ap9" label="TCP" color="blue"></option><option value="HU4agtGOlZ7f" label="SSL" color="blue"></option><option value="pDdUpKMq37rf" label="UDP" color="blue"></option></select></th><th align="center">Use</th></tr></thead><tbody><tr><td align="center">3000</td><td><span data-option="ipZ6Euow0ap9">TCP</span></td><td align="center">Default HTTP port</td></tr><tr><td align="center">4002</td><td><span data-option="HU4agtGOlZ7f">SSL</span></td><td align="center">HTTPS port (encrypted)</td></tr></tbody></table>

[^1]: Default password unless defined in account
