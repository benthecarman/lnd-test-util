# Lnd
> Mostly a copy of [`electrsd`](https://github.com/RCasatta/electrsd) & [`bitcoind`](https://github.com/rust-bitcoin/bitcoind) fit for LND.

Utility to run a regtest [LND](https://github.com/lightningnetwork/lnd) process connected to a given [bitcoind](https://github.com/RCasatta/bitcoind) instance, 
useful in integration testing environment.

```rust
// Returns the ZMQ ports because it is needed for the LND nodes
let (bitcoind, pub_raw_block_port, pub_raw_tx_port) = lnd::setup_bitcoind();

// Pass the binary path, bitcoind, and ZMQ ports
let mut lnd = lnd::Lnd::new("/bin/lnd", &bitcoind, pub_raw_block_port, pub_raw_tx_port);

let node = lnd.client.lightning().get_info(GetInfoRequest {}).await; 
assert!(node.is_ok());
```

## Automatic binaries download

In your project Cargo.toml, activate the following features

```yml
lnd = { git = "https://github.com/bennyhodl/lnd-test-util" }
```

Then use it:

```rust
let bitcoind_exe = bitcoind::downloaded_exe_path().expect("bitcoind version feature must be enabled");
let bitcoind = bitcoind::BitcoinD::new(bitcoind_exe).unwrap();
let lnd_exe = lnd::downloaded_exe_path().expect("lnd version feature must be enabled");
let lnd = lnd::Lnd::new(lnd_exe, bitcoind).unwrap();
```

When the `LND_DOWNLOAD_ENDPOINT`/`BITCOIND_DOWNLOAD_ENDPOINT` environment variables are set,
`lnd`/`bitcoind` will try to download the binaries from the given endpoints.

When you don't use the auto-download feature you have the following options:

- have `lnd` executable in the `PATH`
- provide the `lnd` executable via the `LND_EXEC` env var

```rust
if let Ok(exe_path) = lnd::exe_path() {
  let lnd = lnd::Lnd::new(exe_path, &bitcoind, pub_raw_block_port, pub_raw_tx_port).unwrap();
}
```
## Features

  * lnd use a temporary directory as db dir
  * A free port is asked to the OS (a very low probability race condition is still possible) 
  * The process is killed when the struct goes out of scope no matter how the test finishes

Thanks to these features every `#[test]` could easily run isolated with its own environment
