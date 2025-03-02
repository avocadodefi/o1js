# o1js README-dev

o1js is a TypeScript framework designed for zk-SNARKs and zkApps on the Mina blockchain.

- [zkApps Overview](https://docs.minaprotocol.com/zkapps)
- [Mina README](/src/mina/README.md)

For more information on our development process and how to contribute, see [CONTRIBUTING.md](https://github.com/o1-labs/o1js/blob/main/CONTRIBUTING.md). This document is meant to guide you through building o1js from source and understanding the development workflow.

## Prerequisites

Before starting, ensure you have the following tools installed:

- [Git](https://git-scm.com/)
- [Node.js and npm](https://nodejs.org/)
- [Dune](https://github.com/ocaml/dune) (only needed when compiling o1js from source)
- [Cargo](https://www.rust-lang.org/learn/get-started) (only needed when compiling o1js from source)

After cloning the repository, you need to fetch the submodules:

```sh
git submodule update --init --recursive
```

## Building o1js

For most users, building o1js is as simple as running:

```sh
npm install
npm run build
```

This will compile the TypeScript source files, making it ready for use. The compiled OCaml and WebAssembly artifacts are version-controlled to simplify the build process for end-users. These artifacts are stored under `src/bindings/compiled`, and contain the artifacts needed for both node and web builds. These files do not have to be regenerated unless there are changes to the OCaml or Rust source files.

## Building Bindings

If you need to regenerate the OCaml and WebAssembly artifacts, you can do so within the o1js repo. The [bindings](https://github.com/o1-labs/o1js-bindings) and [Mina](https://github.com/MinaProtocol/mina) repos are both submodules of o1js, so you can build them from within the o1js repo.

o1js depends on OCaml code that is transpiled to JavaScript using [Js_of_ocaml](https://github.com/ocsigen/js_of_ocaml), and Rust code that is transpiled to WebAssembly using [wasm-pack](https://github.com/rustwasm/wasm-pack). These artifacts allow o1js to call into [Pickles](https://github.com/MinaProtocol/mina/blob/develop/src/lib/pickles/README.md), [snarky](https://github.com/o1-labs/snarky), and [Kimchi](https://github.com/o1-labs/proof-systems) to write zk-SNARKs and zkApps.

The compiled artifacts are stored under `src/bindings/compiled`, and are version-controlled to simplify the build process for end-users.

If you wish to rebuild the OCaml and Rust artifacts, you must be able to build the Mina repo before building the bindings. See the [Mina Dev Readme](https://github.com/MinaProtocol/mina/blob/develop/README-dev.md) for more information. Once you have configured your environment to build Mina, you can build the bindings:

```sh
npm run build:update-bindings
```

This will build the OCaml and Rust artifacts, and copy them to the `src/bindings/compiled` directory.

### Build Scripts

The root build script which kicks off the build process is under `src/bindings/scripts/update-snarkyjs-bindings.sh`. This script is responsible for building the Node.js and web artifacts for o1js, and places them under `src/bindings/compiled`, to be used by o1js.

### OCaml Bindings

o1js depends on Pickles, snarky, and parts of the Mina transaction logic, all of which are compiled to JavaScript and stored as artifacts to be used by o1js natively. The OCaml bindings are located under `src/bindings`. See the [OCaml Bindings README](https://github.com/o1-labs/o1js-bindings/blob/main/README.md) for more information.

To compile the OCaml code, a build tool called Dune is used. Dune is a build system for OCaml projects, and is used in addition with Js_of_ocaml to compile the OCaml code to JavaScript. The dune file that is responsible for compiling the OCaml code is located under `src/bindings/ocaml/dune`. There are two build targets: `snarky_js_node` and `snarky_js_web`, which compile the Mina dependencies as well as link the wasm artifacts to build the Node.js and web artifacts, respectively. The output file is `snark_js_node.bc.js`, which is used by o1js.

### WebAssembly Bindings

o1js additionally depends on Kimchi, which is compiled to WebAssembly. Kimchi is located in the Mina repo, under `src/mina`. See the [Kimchi README](https://github.com/o1-labs/proof-systems/blob/master/README.md) for more information.

To compile the wasm code, a combination of Cargo and Dune is used. Both build files are located under `src/mina/src/lib/crypto/kimchi`, where the `wasm` folder contains the Rust code which is compiled to wasm, and the `js` folder which contains a wrapper around the wasm code which allows Js_of_ocaml to compile against the wasm backend.

For the wasm build, the output files are:

- `plonk_wasm_bg.wasm`: The compiled WebAssembly binary.
- `plonk_wasm_bg.wasm.d.ts`: TypeScript definition files describing the types of .wasm or .js files.
- `plonk_wasm.js`: JavaScript file that wraps the WASM code for use in Node.js.
- `plonk_wasm.d.ts`: TypeScript definition file for plonk_wasm.js.

### Generated Constant Types

In addition to building the OCaml and Rust code, the build script also generates TypeScript types for constants used in the Mina protocol. These types are generated from the OCaml source files, and are located under `src/bindings/crypto/constants.ts` and `src/bindings/mina-transaction/gen`. When building the bindings, these constants are auto-generated by Dune. If you wish to add a new constant, you can edit the `src/bindings/ocaml/snarky_js_constants` file, and then run `npm run build:bindings` to regenerate the TypeScript files.

These types are used by o1js to ensure that the constants used in the protocol are consistent with the OCaml source files.

## Development

### Branch Compatibility

If you work on o1js, create a feature branch off of one of these base branches. It's encouraged to submit your work-in-progress as a draft PR to raise visibility! When working with submodules and various interconnected parts of the stack, ensure you are on the correct branches that are compatible with each other.

#### How to Use the Branches

**Default to `main` as the base branch**.

The other base branches (`berkeley`, `develop`) are only used in specific scenarios where you want to adapt o1js to changes in the sibling repos on those other branches. Even then, consider whether it is feasible to land your changes to `main` and merge to `berkeley` and `develop` afterwards. Only changes in `main` will ever be released, so anything in the other branches has to be backported and reconciled with `main` eventually.

| Repository | mina -> o1js -> o1js-bindings    |
| ---------- | -------------------------------- |
| Branches   | o1js-main -> main -> main        |
|            | berkeley -> berkeley -> berkeley |
|            | develop -> develop -> develop    |

- `o1js-main`: The o1js-main branch in the Mina repository corresponds to the main branch in both o1js and o1js-bindings repositories. This is where stable releases and ramp-up features are maintained. The o1js-main branch runs in parallel to the Mina `berkeley` branch and does not have a subset or superset relationship with it. The branching structure is as follows (<- means direction to merge):

  - `develop` <- `o1js-main` <- `current testnet` - Typically, the current testnet often corresponds to the rampup branch.

- `berkeley`: The berkeley branch is maintained across all three repositories. This branch is used for features and updates specific to the Berkeley release of the project.

- `develop`: The develop branch is also maintained across all three repositories. It is used for ongoing development, testing new features, and integration work.

### Running Tests

To ensure your changes don't break existing functionality, run the test suite:

```sh
npm run test
npm run test:unit
```

This will run all the unit tests and provide you with a summary of the test results.

You can additionally run integration tests by running:

```sh
npm run test:integration
```

Finally, we have a set of end-to-end tests that run against the browser. These tests are not run by default, but you can run them by running:

```sh
npm install
npm run e2e:install
npm run build:web

npm run e2e:prepare-server
npm run test:e2e
npm run e2e:show-report
```

### Run the GitHub actions locally

<!-- The test example should stay in sync with a real value set in .github/workflows/build-actions.yml -->

You can execute the CI locally by using [act](https://github.com/nektos/act). First generate a GitHub token and use:

```sh
act -j Build-And-Test-Server --matrix test_type:"Simple integration tests" -s $GITHUB_TOKEN
```

### Releasing

To release a new version of o1js, you must first update the version number in `package.json`. Then, you can create a new pull request to merge your changes into the main branch. Once the pull request is merged, a CI job will automatically publish the new version to npm.

## Test zkApps against the local blockchain network

In order to be able to test zkApps against the local blockchain network, you need to spin up such a network first.  
You can do so in several ways.

1. Using [zkapp-cli](https://www.npmjs.com/package/zkapp-cli)'s sub commands:

   ```shell
   zk lightnet start # start the local network
   # Do your tests and other interactions with the network
   zk lightnet logs # manage the logs of the local network
   zk lightnet explorer # visualize the local network state
   zk lightnet stop # stop the local network
   ```

   Please refer to `zk lightnet --help` for more information.

2. Using the corresponding [Docker image](https://hub.docker.com/r/o1labs/mina-local-network) manually:

   ```shell
   docker run --rm --pull=missing -it \
     --env NETWORK_TYPE="single-node" \
     --env PROOF_LEVEL="none" \
     --env LOG_LEVEL="Trace" \
     -p 3085:3085 \
     -p 5432:5432 \
     -p 8080:8080 \
     -p 8181:8181 \
     -p 8282:8282 \
     o1labs/mina-local-network:o1js-main-latest-lightnet
   ```

   Please refer to the [Docker Hub repository](https://hub.docker.com/r/o1labs/mina-local-network) for more information.

Next up, you will need the Mina blockchain accounts information in order to be used in your zkApp.  
Once the local network is up and running, you can use the [Lightnet](https://github.com/o1-labs/o1js/blob/ec789794b2067addef6b6f9c9a91c6511e07e37c/src/lib/fetch.ts#L1012) `o1js API namespace` to get the accounts information.  
The corresponding example can be found here: [src/examples/zkapps/hello_world/run_live.ts](https://github.com/o1-labs/o1js/blob/ec789794b2067addef6b6f9c9a91c6511e07e37c/src/examples/zkapps/hello_world/run_live.ts)
