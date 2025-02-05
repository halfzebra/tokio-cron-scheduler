# tokio-cron-scheduler

Use cron-like scheduling in an async tokio environment.

Inspired by https://github.com/lholden/job_scheduler

[![](https://docs.rs/tokio_cron_scheduler/badge.svg)](https://docs.rs/tokio_cron_scheduler) [![](https://img.shields.io/crates/v/tokio_cron_scheduler.svg)](https://crates.io/crates/tokio_cron_scheduler) [![](https://travis-ci.org/mvniekerk/tokio_cron_scheduler.svg?branch=master)](https://travis-ci.org/mvniekerk/tokio_cron_scheduler)


## Usage

Please see the [Documentation](https://docs.rs/tokio_cron_scheduler/) for more details.

Be sure to add the job_scheduler crate to your `Cargo.toml`:

```toml
[dependencies]
tokio-cron-scheduler = "*"
```

Creating a schedule for a job is done using the `FromStr` impl for the
`Schedule` type of the [cron](https://github.com/zslayton/cron) library.

The scheduling format is as follows:

```text
sec   min   hour   day of month   month   day of week   year
*     *     *      *              *       *             *
```

Time is specified for `UTC` and not your local timezone. Note that the year may
be omitted.

Comma separated values such as `5,8,10` represent more than one time value. So
for example, a schedule of `0 2,14,26 * * * *` would execute on the 2nd, 14th,
and 26th minute of every hour.

Ranges can be specified with a dash. A schedule of `0 0 * 5-10 * *` would
execute once per hour but only on day 5 through 10 of the month.

Day of the week can be specified as an abbreviation or the full name. A
schedule of `0 0 6 * * Sun,Sat` would execute at 6am on Sunday and Saturday.

A simple usage example:

```rust
use tokio_cron_scheduler::{JobScheduler, JobToRun, Job};

#[tokio::main]
async fn main() {
    let mut sched = JobScheduler::new();

    sched.add(Job::new("1/10 * * * * *", |uuid, l| {
        println!("I run every 10 seconds");
    }).unwrap());

    sched.add(Job::new("1/30 * * * * *", |uuid, l| {
        println!("I run every 30 seconds");
    }).unwrap());
  
    sched.add(
      Job::new_one_shot(Duration::from_secs(18), |_uuid, _l| {
        println!("{:?} I'm only run once", chrono::Utc::now());
      }).unwrap()
    );

    let jj = Job::new_repeated(Duration::from_secs(8), |_uuid, _l| {
      println!("{:?} I'm repeated every 8 seconds", chrono::Utc::now());
    }).unwrap();
    sched.add(jj);

    sched.start().await;
}
```

## Similar Libraries

* [job_scheduler](https://github.com/lholden/job_scheduler) The crate that inspired this one
* [cron](https://github.com/zslayton/cron) the cron expression parser we use.
* [schedule-rs](https://github.com/mehcode/schedule-rs) is a similar rust library that implements it's own cron expression parser.

## License

TokioCronScheduler is licensed under either of

* Apache License, Version 2.0, ([LICENSE-APACHE](LICENSE-APACHE) or
  http://www.apache.org/licenses/LICENSE-2.0)
* MIT license ([LICENSE-MIT](LICENSE-MIT) or
  http://opensource.org/licenses/MIT)

## Contributing

Unless you explicitly state otherwise, any contribution intentionally submitted
for inclusion in the work by you, as defined in the Apache-2.0 license, shall
be dual licensed as above, without any additional terms or conditions.

Please see the [CONTRIBUTING](CONTRIBUTING.md) file for more information.
