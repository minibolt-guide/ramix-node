---
description: >-
  Build your own "DIY" Bitcoin & Lightning node, and other stuff on a Raspberry
  Pi. No need to trust anyone else.
---

# RaMiX

[![Forks](https://img.shields.io/github/forks/minibolt-guide/ramix-node.svg?style=for-the-badge)](https://github.com/minibolt-guide/ramix-node/network/members) [![Stargazers](https://img.shields.io/github/stars/minibolt-guide/ramix-node.svg?style=for-the-badge)](https://github.com/minibolt-guide/ramix-node/stargazers) [![MIT License](https://img.shields.io/github/license/minibolt-guide/ramix-node.svg?style=for-the-badge)](https://raw.githubusercontent.com/minibolt-guide/ramix-node/main/LICENSE) [![Contributors](https://img.shields.io/github/contributors/minibolt-guide/ramix-node.svg?style=for-the-badge)](https://github.com/minibolt-guide/ramix-node/graphs/contributors) [![Issues](https://img.shields.io/github/issues/minibolt-guide/ramix-node.svg?style=for-the-badge)](https://github.com/minibolt-guide/ramix-node/issues) [![GitHub release (latest by date)](https://img.shields.io/github/v/release/minibolt-guide/ramix-node?label=latest%20release)](https://github.com/minibolt-guide/ramix-node/releases) [![GitHub followers](https://img.shields.io/github/followers/minibolt-guide)](https://github.com/orgs/minibolt-guide/followers) [![GitBook](https://img.shields.io/static/v1?message=Documented%20on%20GitBook\&logo=gitbook\&logoColor=ffffff\&label=%20\&labelColor=5c5c5c\&color=3F89A1)](https://www.gitbook.com/preview?utm_source=gitbook_readme_badge\&utm_medium=organic\&utm_campaign=preview_documentation\&utm_content=link)

{% hint style="danger" %}
<mark style="color:red;">**Attention!! This guide is in the WIP (work in progress) state and hasn't been tested yet. Many steps may be incorrect. Pay special attention to the "Status: Not tested on RaMiX" tag at the beginning of the guides. Be careful and act behind your responsibility.**</mark>
{% endhint %}

<figure><img src=".gitbook/assets/home_screen.webp" alt=""><figcaption></figcaption></figure>

## What is the RaMiX?

With this guide, you can set up a Bitcoin, Lightning node, and other stuff on a Raspberry Pi, doing everything yourself. You will learn about Linux, Bitcoin, Lightning, and much more.

<figure><img src=".gitbook/assets/tgfoss-build-under-win.gif" alt=""><figcaption></figcaption></figure>

There are many reasons why you should run your own Bitcoin node:

👥 **Keep Bitcoin decentralized:** use your node to help enforce your Bitcoin consensus rules.

🗽 **Take back your sovereignty:** let your node validate your Bitcoin transactions. No need to ask someone else to tell you what's happening in the Bitcoin network.

🥷🏽 **Improve your privacy:** connect your wallets to your node so you no longer need to reveal your financial history to external servers.

⚡️ **Be part of Lightning:** run your Lightning node for everyday payments and help build a robust and decentralized Lightning network.

## RaMiX overview

This guide explains setting up your Bitcoin node on a Raspberry Pi. However, it works on most hardware platforms because it only uses standard Debian-based Linux commands.

### Features

Your Bitcoin node will offer the following functionality:

🟠 **Bitcoin**: direct and trustless participation in the Bitcoin peer-to-peer network, full validation of blocks and transactions

⚛️ **Electrum server**: connect your compatible wallets (including hardware wallets) to your node

⛓️ **Blockchain Explorer**: web-based Explorer to privately look up transactions, blocks, and more

⚡ **Lightning**: full client with stable long-term channels and web-based and mobile-based management interfaces

🔋 **Always on**: services are constantly synced and available 24/7

🌐 **Reachable from anywhere**: connect to all your services through the Tor network and Wireguard VPN

### Target audience

* [x] We strive to give foolproof instructions. But the goal is also to do everything ourselves.
* [x] Shortcuts that involve trusting someone else are not allowed. This makes this guide quite technical, but we try to make it as straightforward as possible.
* [x] You'll gain a basic understanding of the how and why.
* [x] If you want to learn about Linux, Bitcoin, and Lightning, this guide is for you.

### Structure

We aim to keep the core of this guide well-maintained and up-to-date:

<table data-view="cards" data-full-width="false"><thead><tr><th></th><th align="center"></th><th align="center"></th><th data-type="content-ref"></th><th data-type="content-ref"></th><th data-type="content-ref"></th><th data-hidden data-card-target data-type="content-ref"></th><th data-hidden data-card-cover data-type="files"></th></tr></thead><tbody><tr><td><strong>🖥️</strong> <a href="broken-reference/"><strong>System</strong></a></td><td align="center">Prepare the hardware and set up the operating system</td><td align="center"></td><td><a href="index-1/operating-system.md">operating-system.md</a></td><td><a href="index-1/remote-access.md">remote-access.md</a></td><td></td><td><a href="system/">system</a></td><td><a href=".gitbook/assets/operating-system.gif">operating-system.gif</a></td></tr><tr><td><strong>🟠</strong> <a href="broken-reference/"><strong>₿itcoin</strong></a></td><td align="center">Sync your own Bitcoin full node, Electrum server, Blockchain Explorer, and connect a desktop wallet to the Electrum server</td><td align="center"></td><td><a href="bitcoin/bitcoin/electrum-server.md">electrum-server.md</a></td><td><a href="bitcoin/bitcoin/blockchain-explorer.md">blockchain-explorer.md</a></td><td></td><td><a href="bitcoin/bitcoin/">bitcoin</a></td><td><a href=".gitbook/assets/core_logo.png">core_logo.png</a></td></tr><tr><td><strong>⚡</strong> <a href="broken-reference/"><strong>Lightning</strong></a></td><td align="center">Run your Lightning client with web-based node management, connect a mobile app, and save safely your SCB backup</td><td align="center"></td><td><a href="lightning/channel-backup.md">channel-backup.md</a></td><td><a href="lightning/web-app.md">web-app.md</a></td><td></td><td><a href="lightning/">lightning</a></td><td><a href="images/lightning-network-daemon-logo.png">lightning-network-daemon-logo.png</a></td></tr><tr><td><br><strong>➕</strong> <a href="broken-reference/"><strong>Bonus guides</strong></a><br></td><td align="center">The bonus section contains more specific guides that build on top of the main section. More fun, lots of knowledge, but with lesser maintenance guarantees. Everything is optional.</td><td align="center"></td><td><a href="bonus/system/">system</a></td><td><a href="bonus/bitcoin/">bitcoin</a></td><td><a href="bonus-guides/nostr/">nostr</a></td><td><a href="broken-reference/">broken-reference</a></td><td><a href=".gitbook/assets/bonus-logo.png">bonus-logo.png</a></td></tr></tbody></table>

## Community

<table data-card-size="large" data-view="cards" data-full-width="false"><thead><tr><th align="center"></th><th></th><th></th></tr></thead><tbody><tr><td align="center"><strong>👥 RRSS 👥</strong></td><td></td><td><ul><li><p>Telegram Groups:</p><ul><li><a href="https://t.me/minibolt">English</a></li><li><a href="https://t.me/minibolt_es">Spanish</a></li></ul></li></ul><ul><li><a href="https://habla.news/c/naddr1qqyy66twd9px7mr5qyf8wumn8ghj7mmxve3ksctfdch8qatzqgstzl7vmurm5gu87qutx3pxwgxddrg39huj809zhmv03scfkus3z4grqsqqpphk2j0aff">Nostr community</a></li><li><p>Nostr channels:</p><ul><li><a href="https://www.nostrchat.io/channel/aa64f2ead929ce8417f85bde7d22ebde13cc01ceb4e00145572437eb1ad46249">English</a></li><li><a href="https://www.nostrchat.io/channel/3bd633eaad12242572bfc5ba10d3e52b2c0e152f4207383858993c373d314015">Spanish</a></li></ul></li><li><p>Telegram Channels (News):</p><ul><li><a href="https://t.me/minibolt_news">English</a></li><li><a href="https://t.me/minibolt_es_noticias">Spanish</a></li></ul></li></ul></td></tr><tr><td align="center"><strong>🛠️</strong> <a href="https://github.com/minibolt-guide/minibolt"><strong>GitHub</strong></a> <strong>🛠️</strong></td><td></td><td><ul><li><a href="https://github.com/minibolt-guide/ramix-node/pulls">Pull requests</a></li><li><a href="https://github.com/minibolt-guide/ramix-node/issues">Issues / Knowledge base</a></li><li><a href="https://github.com/minibolt-guide/ramix-node/discussions">Discussions</a></li></ul></td></tr></tbody></table>

{% hint style="info" %}
Feel free to join the many other contributors if you see something that can be improved!
{% endhint %}

## Resources

<table data-view="cards"><thead><tr><th></th><th data-hidden data-card-cover data-type="files"></th><th data-hidden data-card-target data-type="content-ref"></th></tr></thead><tbody><tr><td>🌐 <a href="https://app.diagrams.net/?tags={}&#x26;lightbox=1&#x26;target=blank&#x26;highlight=0000ff&#x26;edit=_blank&#x26;layers=1&#x26;nav=1&#x26;title=networkmap.drawio.png#Uhttps://raw.githubusercontent.com/minibolt-guide/minibolt/main/resources/networkmap.drawio.png">Network map</a></td><td><a href=".gitbook/assets/networkmap_icon.png">networkmap_icon.png</a></td><td><a href="https://bit.ly/minibolt-ramix_netmap">https://bit.ly/minibolt-ramix_netmap</a></td></tr><tr><td>🛣️ <a href="https://github.com/orgs/minibolt-guide/projects/1">Roadmap</a></td><td><a href=".gitbook/assets/roadmap.png">roadmap.png</a></td><td><a href="https://github.com/orgs/minibolt-guide/projects/1">https://github.com/orgs/minibolt-guide/projects/1</a></td></tr><tr><td>📅 <a href="https://www.flockstr.com/calendar/naddr1qqyrgdmpvvmxxcfjqgstzl7vmurm5gu87qutx3pxwgxddrg39huj809zhmv03scfkus3z4grqsqqql95pq28q6">Launchpad (Calendar)</a></td><td><a href=".gitbook/assets/calendar.png">calendar.png</a></td><td></td></tr><tr><td>📺 <a href="https://zap.stream/p/npub1k9luehc8hg3c0upckdzzvusv66x3zt0eyw7290kclrpsndepz92sfcpp63">Streams</a></td><td><a href=".gitbook/assets/streams.png">streams.png</a></td><td></td></tr></tbody></table>

## Free services

{% tabs %}
{% tab title="Nostr relay" %}
{% hint style="info" %}
Use [a Nostr client](bonus-guides/nostr/nostr-relay.md#nostr-clients) to connect
{% endhint %}

* Nostr relay in Rust (🚾clearnet):

<pre class="language-url"><code class="lang-url"><strong>wss://relay.minibolt.info
</strong></code></pre>

* Nostr relay in Rust (🧅onion):

```url
ws://xib7qeea6f5nz3ueko4kwcsddpvggdray4nhagcvofbioot3n2qrapid.onion
```
{% endtab %}

{% tab title="Electrum server" %}
{% hint style="info" %}
Use a signing app ([Sparrow Wallet](bitcoin/bitcoin/desktop-signing-app-sparrow.md) or [Electrum Wallet desktop](bonus/bitcoin/electrum-wallet-desktop.md)) to connect
{% endhint %}

* Fulcrum - mainnet (🧅onion):

```url
tcp://vr4bgiwqlhuweftttc6bj7lm5ijjyafwsr43nmeiu3k4mcgtl4tpozyd.onion:50001
```

* Fulcrum - testnet4 (🧅onion):

```url
tcp://bnfpvanrc2g7r5o5kaabbbyjv6ddh46jmasfatrvbbsvjb7cdik5n7ad.onion:40001
```

```url
ssl://bnfpvanrc2g7r5o5kaabbbyjv6ddh46jmasfatrvbbsvjb7cdik5n7ad.onion:40002
```
{% endtab %}

{% tab title="Explorer" %}
* BTC RPC Explorer - **mainnet** (🚾clearnet):

{% hint style="danger" %}
Temporary not accessible from EE.UU, Austria, Singapore, or Germany
{% endhint %}

&#x20;👉 CLICK to access: [https://explorer.minibolt.info](https://explorer.minibolt.info) 👈

* BTC RPC Explorer - **mainnet** (🧅onion - Use [Tor browser](https://www.torproject.org/download/)):

{% hint style="success" %}
Accessible from **anywhere**
{% endhint %}

```url
http://rzcj4r2p6wterkto5prigsplq6iva5bqhcxr7y3d6w4hoc3uwizpp5qd.onion
```
{% endtab %}

{% tab title="Keyserver" %}
* Hockeypuck OpenPGP Public Keyserver (🚾clearnet):

👉 CLICK to access: [https://keyserver.minibolt.info](https://keyserver.minibolt.info) 👈

* Hockeypuck OpenPGP Public Keyserver (🧅onion - use [Tor browser](https://www.torproject.org/download/)):

```
http://fr2bbk7gitvpielymw7jmbkmm7glrzs2avxyxsh3rqbszkwavmqkklid.onion
```
{% endtab %}
{% endtabs %}

## Rating

All guides are rated with labels to help you assess their difficulty and whether they are tested against the most recent version of the main guide.

* **Difficulty:** indicates how difficult the bonus guide is in terms of installation procedure or usage

{% hint style="success" %}
Difficulty: Easy
{% endhint %}

{% hint style="warning" %}
Difficulty: Medium
{% endhint %}

{% hint style="danger" %}
Difficulty: Hard
{% endhint %}

* **Cost:** indicates if the service used in the guide is free or paid

{% hint style="warning" %}
Cost: Paid service
{% endhint %}

* **Status:** indicates if the guide has been tested on a RaMiX environment or not

{% hint style="danger" %}
Status: Not tested on RaMiX
{% endhint %}

## Port reference

<table><thead><tr><th align="center">Port</th><th width="100">Protocol<select><option value="ZziTt9Pqfrg7" label="TCP" color="blue"></option><option value="p7cIpyRs6gED" label="SSL" color="blue"></option><option value="9iVSfYOS7FjP" label="UDP" color="blue"></option></select></th><th align="center">Use</th></tr></thead><tbody><tr><td align="center">🖥️ SYSTEM</td><td></td><td align="center"></td></tr><tr><td align="center">22</td><td><span data-option="ZziTt9Pqfrg7">TCP</span></td><td align="center">Default SSH server port</td></tr><tr><td align="center">9050</td><td><span data-option="ZziTt9Pqfrg7">TCP</span></td><td align="center">Default Tor SOCKS port</td></tr><tr><td align="center">9051</td><td><span data-option="ZziTt9Pqfrg7">TCP</span></td><td align="center">Default Tor control port</td></tr><tr><td align="center">7656</td><td><span data-option="ZziTt9Pqfrg7">TCP</span></td><td align="center">Default I2P SAM proxy port</td></tr><tr><td align="center">7070</td><td><span data-option="ZziTt9Pqfrg7">TCP</span></td><td align="center">Default I2Pd web console port</td></tr><tr><td align="center">7071</td><td><span data-option="p7cIpyRs6gED">SSL</span></td><td align="center">I2Pd web console SSL port</td></tr><tr><td align="center">🟠 ₿ITCOIN</td><td></td><td align="center"></td></tr><tr><td align="center">8332</td><td><span data-option="ZziTt9Pqfrg7">TCP</span></td><td align="center">Default Bitcoin Core RPC port</td></tr><tr><td align="center">8333</td><td><span data-option="ZziTt9Pqfrg7">TCP</span></td><td align="center">Default Bitcoin Core P2P port</td></tr><tr><td align="center">8334</td><td><span data-option="ZziTt9Pqfrg7">TCP</span></td><td align="center">Default Bitcoin Core P2P Tor port</td></tr><tr><td align="center">50001</td><td><span data-option="ZziTt9Pqfrg7">TCP</span></td><td align="center">Fulcrum TCP port</td></tr><tr><td align="center">50002</td><td><span data-option="p7cIpyRs6gED">SSL</span></td><td align="center">Fulcrum SSL port</td></tr><tr><td align="center">8000</td><td><span data-option="ZziTt9Pqfrg7">TCP</span></td><td align="center">Fulcrum Admin port</td></tr><tr><td align="center">3002</td><td><span data-option="ZziTt9Pqfrg7">TCP</span></td><td align="center">Default BTC RPC Explorer HTTP port</td></tr><tr><td align="center">4000</td><td><span data-option="p7cIpyRs6gED">SSL</span></td><td align="center">BTC RPC Explorer HTTPS port (encrypted)</td></tr><tr><td align="center">⚡ LIGHTNING</td><td></td><td align="center"></td></tr><tr><td align="center">9735</td><td><span data-option="ZziTt9Pqfrg7">TCP</span></td><td align="center">Default LND P2P port</td></tr><tr><td align="center">10009</td><td><span data-option="ZziTt9Pqfrg7">TCP</span></td><td align="center">Default LND gRPC port</td></tr><tr><td align="center">9911</td><td><span data-option="ZziTt9Pqfrg7">TCP</span></td><td align="center">Default LND Watchtower server port</td></tr><tr><td align="center">3000</td><td><span data-option="ZziTt9Pqfrg7">TCP</span></td><td align="center">Default ThunderHub HTTP port</td></tr><tr><td align="center">4002</td><td><span data-option="p7cIpyRs6gED">SSL</span></td><td align="center">ThunderHub HTTPS port (encrypted)</td></tr><tr><td align="center">8080</td><td><span data-option="ZziTt9Pqfrg7">TCP</span></td><td align="center">LND REST port</td></tr><tr><td align="center">➕ BONUS GUIDES</td><td></td><td align="center"></td></tr><tr><td align="center">5432</td><td><span data-option="ZziTt9Pqfrg7">TCP</span></td><td align="center">Default PostgreSQL relational DB port</td></tr><tr><td align="center">51820</td><td><span data-option="9iVSfYOS7FjP">UDP</span></td><td align="center">Default WireGuard VPN port</td></tr><tr><td align="center">Random</td><td><span data-option="ZziTt9Pqfrg7">TCP</span></td><td align="center">Random Cloudflared port</td></tr><tr><td align="center">&#x3C;TODO1></td><td><span data-option="ZziTt9Pqfrg7">TCP</span></td><td align="center">obfs4 bridge OR port</td></tr><tr><td align="center">&#x3C;TODO2></td><td><span data-option="ZziTt9Pqfrg7">TCP</span></td><td align="center">obfs4 port</td></tr><tr><td align="center">9001</td><td><span data-option="ZziTt9Pqfrg7">TCP</span></td><td align="center">OR port Guard/Middle relay</td></tr><tr><td align="center">9052</td><td><span data-option="ZziTt9Pqfrg7">TCP</span></td><td align="center">Tor obfs4 bridge control port</td></tr><tr><td align="center">9053</td><td><span data-option="ZziTt9Pqfrg7">TCP</span></td><td align="center">Tor Guard/Middle relay control port</td></tr><tr><td align="center">50021</td><td><span data-option="ZziTt9Pqfrg7">TCP</span></td><td align="center">Electrs TCP port</td></tr><tr><td align="center">50022</td><td><span data-option="p7cIpyRs6gED">SSL</span></td><td align="center">Electrs SSL port</td></tr><tr><td align="center">48333</td><td><span data-option="ZziTt9Pqfrg7">TCP</span></td><td align="center">Default Bitcoin Core P2P Testnet4 port</td></tr><tr><td align="center">48334</td><td><span data-option="ZziTt9Pqfrg7">TCP</span></td><td align="center">Default Bitcoin Core P2P Testnet4 secondary port</td></tr><tr><td align="center">48332</td><td><span data-option="ZziTt9Pqfrg7">TCP</span></td><td align="center">Default Bitcoin Core RPC Testnet4 port</td></tr><tr><td align="center">40001</td><td><span data-option="ZziTt9Pqfrg7">TCP</span></td><td align="center">Fulcrum Testnet4 port</td></tr><tr><td align="center">40002</td><td><span data-option="p7cIpyRs6gED">SSL</span></td><td align="center">Fulcrum Testnet4 SSL port</td></tr><tr><td align="center">40021</td><td><span data-option="ZziTt9Pqfrg7">TCP</span></td><td align="center">Electrs Testnet4 port</td></tr><tr><td align="center">40022</td><td><span data-option="p7cIpyRs6gED">SSL</span></td><td align="center">Electrs server Testnet4 SSL port</td></tr><tr><td align="center">24444</td><td><span data-option="ZziTt9Pqfrg7">TCP</span></td><td align="center">Default NBXplorer port</td></tr><tr><td align="center">23000</td><td><span data-option="ZziTt9Pqfrg7">TCP</span></td><td align="center">Default BTCPay Server port</td></tr><tr><td align="center">8880</td><td><span data-option="ZziTt9Pqfrg7">TCP</span></td><td align="center">Default Nostr relay port</td></tr></tbody></table>
