# Linux development environment

Rust supports most Linux distributions.
Depending on the specific distribution and version of the operating system you use, you might need to add some package dependencies to your environment.
In general, your development environment should include a linker or C-compatible compiler such as `clang` and an appropriate integrated development environment (IDE).

## Before you begin

Check the documentation for your operating system for information about the packages that are installed and how to download and install any additional packages you might need. 
For example, if you use Ubuntu, you can use the Ubuntu Advanced Packaging Tool (`apt`) to install the `build-essential` package:

```bash
sudo apt install build-essential
```

At a minimum, you need the following packages before you install Rust:

```bash
clang
curl
git
```

Because the blockchain requires standard cryptography to support the generation of public/private key pairs and the validation of transaction signatures, you must also have a package that provides cryptography, such as `libssl-dev` or `openssl-devel`.

## Install required packages and Rust

To install the Rust toolchain on Linux:

1. Log on to your computer and open a terminal shell.

1. Check the packages you have installed on the local computer by running an appropriate package management command for your Linux distribution.

1. Add any package dependencies you are missing to your local development environment by running an appropriate package management command for your Linux distribution.
    
    For example, on Ubuntu, you might run a command similar to the following:
    
    ```bash
    sudo apt install --assume-yes git clang curl libssl-dev
    ```

1. Download the `rustup` installation program and use it to install Rust by running the following command:
    
    ```bash
    curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
    ```

1. Follow the prompts displayed to proceed with a default installation.

1. Update your current shell to include Cargo by running the following command:
    
    ```bash
    source $HOME/.cargo/env
    ```

1. Verify your installation by running the following command:
    
    ```bash
    rustc --version
    ```

1. Configure the Rust toolchain to default to the latest stable version by running the following commands:
    
    ```bash
    rustup default stable
    rustup update
    ```

1. Add the `nightly` release and the `nightly` WebAssembly (wasm) targets to your development environment by running the following commands:
    
    ```bash
    rustup update nightly
    rustup target add wasm32-unknown-unknown --toolchain nightly
    ```

1. Verify the configuration of your development environment by running the following command:
    
    ```bash
    rustup show
    ```

    The command displays output similar to the following:

    <pre>
    Default host: x86_64-unknown-linux-gnu
    rustup home:  /home/subdocs/.rustup
    
    installed toolchains
    --------------------
    
    stable-x86_64-unknown-linux-gnu (default)
    nightly-x86_64-unknown-linux-gnu
    
    active toolchain
    ----------------
    
    stable-x86_64-unknown-linux-gnu (default)
    rustc 1.58.1 (db9d1b20b 2022-01-20)
    </pre>

## Compile a Substrate node

Now that you have Rust installed and the Rust toolchains configured for Substrate development, you are ready to finish setting up your development environment by cloning the Substrate **node template** files and compiling a Substrate node.

The node template provides a working environment that includes all of the most common features you need to build a blockchain without any extraneous modules or tools. 
To ensure that the node template offers a relatively stable working environment for you to experiment with, the recommended best practice is to clone Substrate node template from the Substrate Developer Hub  repository, rather than from the core Substrate repository.

To compile the Substrate node template:

1. Clone the node template repository by running the following command:
    
    ```bash
    git clone https://github.com/substrate-developer-hub/substrate-node-template
    ```

1. Change to the root of the node template directory by running the following command:
    
    ```bash
    cd substrate-node-template
    ```

1. Switch to the version of the repository that has the `latest` tag by running the following command:
    
    ```bash
    git checkout latest
    ```

1. Compile the node template by running the following command:
    
    ```bash
    cargo build --release
    ```

    Because of the number of packages required, compiling the node can take several minutes.

After the build completes successfully, your local computer is ready for Substrate development activity.
