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
async-periodic-job = "0.1.2"
```

### Usage

```rust
use async_periodic_job::{Job, Scheduler};
use std::time::Duration;
use tokio::time::sleep;
use tokio_util::sync::CancellationToken;

// Define a job without options
struct JobA;
impl Job for JobA {
    async fn run(&mut self) {
        println!("JobA run");
    }
}

// Define a job with options
struct JobB;
impl Job for JobB {
    // Job repeat period, default: 1s
    fn period(&self) -> Duration {
        Duration::from_secs(2)
    }

    // Job run time truncation, default: true
    fn truncate_time(&self) -> bool {
        true
    }

    // Job run
    async fn run(&mut self) {
        println!("JobB run");
    }
}

#[tokio::main]
async fn main() {
    // Spawn jobs
    let scheduler = Scheduler::new().spawn(JobA).spawn(JobB);

    // Wait scheduler stop
    // Schedules will be stopped after received ctrl_c signal
    scheduler.wait().await;
}

// Schedule jobs with cancel token
async fn cancel_example() {
    // Spawn jobs
    let scheduler = Scheduler::new().spawn(JobA).spawn(JobB);

    // Create a cancellation token
    let token = CancellationToken::new();

    // Spawn a future to cancel the token after 5secs
    let token_c = token.clone();
    tokio::spawn(async move {
        sleep(Duration::from_secs(5)).await;
        token_c.cancel();
    });

    // Wait scheduler stop
    // Scheduler will be stopped after the token cancelled
    scheduler.wait_cancel(token).await;
}

```

## License

MIT

Contributions and suggestions are welcome!
