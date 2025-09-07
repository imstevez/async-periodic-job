# async-periodic-job

[![Crates.io](https://img.shields.io/crates/v/async-periodic-job)](https://crates.io/crates/async-periodic-job)
[![docs](https://img.shields.io/crates/v/async-periodic-job?color=orange&label=docs)](https://docs.rs/async-periodic-job)

A simple async periodic job scheduler library base on tokio-util.

## Features

- **Periodic Job Execution**: Define jobs that run at regular intervals
- **Asynchronous Execution**: Based on Tokio, schedule jobs in a non-blocking manner
- **Truncated Run Time**: Jobs can be configured to run at truncated time intervals, ensuring precise timing
- **Graceful Shutdown**: Gracefully stop the scheduler using either a `Ctrl+C` signal or a cancellation token

## Quick Started

### Installation

```toml
[dependencies]
async-periodic-job = "0.1.3"
```

### Usage

#### Base Usage

```rust
use async_periodic_job::{Job, Scheduler};
use std::time::Duration;

// Define a job without options
struct JobImplA;
impl Job for JobImplA {
    async fn run(&mut self) {
        // ...
    }
}

// Define a job with customized period and time truncation
struct JobImplB;
impl Job for JobImplB {
    // Job repeat period, default: 1s
    fn period(&self) -> Duration {
        Duration::from_secs(2)
    }

    // If run job with truncate time, default: true
    fn with_truncate_time(&self) -> bool {
        false
    }

    // Job run
    async fn run(&mut self) {
        // ...
    }
}

#[tokio::main]
async fn main() {
    // Spawn job instances of JobA and JobB and wait scheduler stop
    // When received `CTRL+C` signal, scheduler will exit (after all running jobs exit)
    Scheduler::new()
        .spawn(JobImplA)
        .spawn(JobImplB)
        .wait()
        .await;
}

```

#### Scheduler cancellation

```rust
use async_periodic_job::{Job, Scheduler, Token};
use std::time::Duration;
use tokio::time::sleep;

// Define a job with cancel
struct JobImpl;
impl Job for JobImpl {
    async fn run(&mut self) {
        // ...
    }
}

#[tokio::main]
async fn main() {
    // Create a cancellation token and spawn a future to cancel the token after 5 secs
    let token = Token::new();
    let token_copy = token.clone();
    tokio::spawn(async move {
        sleep(Duration::from_secs(5)).await;
        token_copy.cancel();
    });

    // Spawn instance of JobImpl and wait scheduler stop
    // When token cancelled, scheduler will exit (after all running jobs exit)
    Scheduler::new()
        .spawn(JobImpl)
        .wait_cancel(token)
        .await;
}
```

#### Spawn job with cancel

```rust
use async_periodic_job::{Job, Scheduler, Token};

// Define a job with cancel
struct JobImpl;
impl Job for JobImpl {
    // If run job with cancel, default: false
    // When this method returned true, job method `run_with_cancel` will be executed instead of `run`
    fn with_with_cancel(&self) -> bool {
        true
    }

    // Job run: instead implementing `run`, implement `run_with_cancel`
    async fn run_with_cancel(&mut self, token: Token) {
        loop {
            if token.is_cancelled() {
                return;
            }
            // ...
        }
    }
}

#[tokio::main]
async fn main() {
    Scheduler::new()
        .spawn(JobImpl)
        .wait()
        .await;
}
```

## License

MIT

Contributions and suggestions are welcome!
