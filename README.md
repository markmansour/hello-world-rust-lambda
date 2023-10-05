# "Hello, World" in Rust, deployed to AWS Lambda

## Goals
To learn about:
* Rust - how to build a hello world application.  The basics of the Rust toolchain (cargo, etc)
* How to deploy a non-standard language to Lambda

Disclaimer: This is my first Rust project (I wrote my first Hello World Rust program only a few hours before starting this).  This is a learning project.  This isn't an example of my finest coding or my amazing documentation skills.  This project is for me.

## Helpful posts
* [Rust Runtime for AWS Lambda](https://aws.amazon.com/blogs/opensource/rust-runtime-for-aws-lambda/)
* [Lambda execution role](https://docs.aws.amazon.com/lambda/latest/dg/lambda-intro-execution-role.html)

## Getting Started
### Prereqs.

1. Install [Rust](https://www.rust-lang.org/tools/install)/Cargo.
2. Install the [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html).  Run `aws configure`.  See "Creating an execution role in the IAM console" for details on creating the appropriate role.

### Generate scaffolding
Use <code>Cargo</code> to generate scaffolding.

```bash
cargo new hello-world-rust-lambda
```

### Update the code

Update src/main.rs

```rust
use lambda_runtime::{service_fn, LambdaEvent, Error};
use serde_json::{json, Value};

#[tokio::main]
async fn main() -> Result<(), Error> {
    let func = service_fn(func);
    lambda_runtime::run(func).await?;
    Ok(())
}

async fn func(event: LambdaEvent<Value>) -> Result<Value, Error> {
    let (event, _context) = event.into_parts();
    let first_name = event["firstName"].as_str().unwrap_or("world");

    Ok(json!({ "message": format!("Hello, {}!", first_name) }))
}
```

### Update the dependencies
Update `Cargo.toml`

```TOML
[dependencies]
lambda_runtime = "0.8.2"
serde_json = "1.0.107"
tokio = "1.32.0"
```

## Build
Build for AWS servers / [X86 Linux](https://doc.rust-lang.org/rustc/platform-support.html) (not my Apple M2 mac).  By default it builds for X86.

```bash
cargo lambda build --release --output-format zip
```

Deploy the function to the Lambda service:

```bash
aws lambda create-function --function-name hello-rust-world --handler bootstrap --zip-file fileb://./target/lambda/hello-world-rust-lambda/bootstrap.zip --runtime provided.al2 --role arn:aws:iam::264225025302:role/lambda_execution --environment Variables={RUST_BACKTRACE=1} --tracing-config Mode=Active
```

Test the invocation from the CLI:

```bash
aws lambda invoke --cli-binary-format raw-in-base64-out --function-name hello-rust-world --payload '{"firstName": "Mark"}' output.json
```
