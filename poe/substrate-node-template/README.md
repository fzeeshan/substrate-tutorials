# STOP!!!
Read this before starting: This repo was cloned from tutorials/solutions/proof-of-existence branch from the main repository which is as follows"https://github.com/substrate-developer-hub/substrate-node-template"
This branch covers Proof of Existence tutorial and has the solution already implemented. 

Idea is to have a working repository that can work for new developers trying to learn Substrate framework.
# Updated Installation
Follow standard installation of rust and related tools. For this tutorial you will need specific rust versions.
# Use Compatible Rust Install
Check to see which rust versions you have installed on your system
```sh
rustup toolchain list
```
Install the correct version which is as follows:
generally toolchain would be installed with rust, however, i have also added the command to install it below:
```sh
rustup install nightly-2023-06-15
rustup toolchain install nightly-2023-06-15-x86_64-unknown-linux-gnu
rustup target add wasm32-unknown-unknown --toolchain nightly-2023-06-15-x86_64-unknown-linux-gnu
```


### Build

Use the following command to build the node without launching it:
Also Included check command
```sh
cargo +nightly-2023-06-15 check -p node-template-runtime --release
cargo +nightly-2023-06-15 build --release

```

### Embedded Docs

After you build the project, you can use the following command to explore its parameters and subcommands:

```sh
./target/release/node-template --dev --rpc-external

```
## connect using the front-end or polka explorer
https://polkadot.js.org/apps/?rpc=ws%3A%2F%2F127.0.0.1%3A9944#/explorer

### See the original repo for other options, idea of this repo is to build everything without breaking.
