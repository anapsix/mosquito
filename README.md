
<img src="logo/logotype_horizontal.svg" alt="mosquito">

![GitHub Workflow Status](https://img.shields.io/github/workflow/status/robacarp/mosquito/Test%20and%20Demo?style=for-the-badge)
[![Crystal Version](https://img.shields.io/badge/crystal-0.35.1-blueviolet.svg?longCache=true&style=for-the-badge)](https://crystal-lang.org/)
[![GitHub](https://img.shields.io/github/license/robacarp/mosquito.svg?style=for-the-badge)](https://tldrlegal.com/license/mit-license)

Mosquito is a generic background job runner written specifically for Crystal. Significant inspiration from my experience with the successes and failings of the Ruby gem Sidekiq.

Mosquito currently provides these features:
- Delayed execution
- Scheduled / Periodic execution
- Job Storage in Redis
- Crystal hash style `?` methods for parameter getters which return nil instead of raise
- Automatic rescheduling of failed jobs
- Progressively increasing delay of failed jobs
- Dead letter queue of jobs which have failed too many times
- Rate limited jobs

Current Limitations:
- Job failure delay, maximum retry count, and several other variables cannot be easily configured.
- Visibility into the job queue is difficult and must be done through redis manually.

![](https://cdn.shopify.com/s/files/1/0242/0179/products/amber1_1024x1024.png?v=1455409061)

## Project State

Updated 2020-10-26

> Stable.
>
> A few projects are using Mosquito in production, and it's going okay.
>
> If you're using Mosquito, please get in touch.

## Installation

Update your `shard.yml` to include mosquito:

```diff
dependencies:
+  mosquito:
+    github: robacarp/mosquito
```

Further installation instructions are available for use with Amber as well as a vanilla crystal application:

- [Installing with Amber](https://github.com/robacarp/mosquito/wiki/Usage:-Amber)
- [Adding to a vanilla crystal application](https://github.com/robacarp/mosquito/wiki/Usage:-vanilla-crystal)

## Usage

### Step 1: Define a queued job

```crystal
# src/jobs/puts_job.cr
class PutsJob < Mosquito::QueuedJob
  params message : String

  def perform
    puts message
  end
end
```

### Step 2: Trigger that job

```crystal
# src/<somewher>/<somefile>.cr
PutsJob.new(message: "ohai background job").enqueue
```

### Step 3: Run your worker to process the job

```crystal
# src/worker.cr

Mosquito.configure do |settings|
  settings.redis_url = ENV["REDIS_URL"]
end

Mosquito::Runner.start
```

```text
crystal run src/worker.cr
```

### Success

```
> crystal run src/worker.cr
2017-11-06 17:07:29 - Mosquito is buzzing...
2017-11-06 17:07:51 - Running task puts_job<...> from puts_job
2017-11-06 17:07:51 - [PutsJob] ohai background job
2017-11-06 17:07:51 - task puts_job<...> succeeded, took 0.0 seconds
```

[More information about queued jobs](https://github.com/robacarp/mosquito/wiki/Queued-jobs) in the wiki.

------

## Periodic Jobs

Periodic jobs run according to a predefined period. 

This periodic job:
```crystal
class PeriodicallyPutsJob < Mosquito::PeriodicJob
  run_every 1.minute

  def perform
    emotions = %w{happy sad angry optimistic political skeptical epuhoric}
    puts "The time is now #{Time.local} and the wizard is feeling #{emotions.sample}"
  end
end
```

Would produce this output:
```crystal
2017-11-06 17:20:13 - Mosquito is buzzing...
2017-11-06 17:20:13 - Queues: periodically_puts_job
2017-11-06 17:20:13 - Running task periodically_puts_job<...> from periodically_puts_job
2017-11-06 17:20:13 - [PeriodicallyPutsJob] The time is now 2017-11-06 17:20:13 and the wizard is feeling skeptical
2017-11-06 17:20:13 - task periodically_puts_job<...> succeeded, took 0.0 seconds
2017-11-06 17:21:14 - Queues: periodically_puts_job
2017-11-06 17:21:14 - Running task periodically_puts_job<...> from periodically_puts_job
2017-11-06 17:21:14 - [PeriodicallyPutsJob] The time is now 2017-11-06 17:21:14 and the wizard is feeling optimistic
2017-11-06 17:21:14 - task periodically_puts_job<...> succeeded, took 0.0 seconds
2017-11-06 17:22:15 - Queues: periodically_puts_job
2017-11-06 17:22:15 - Running task periodically_puts_job<...> from periodically_puts_job
2017-11-06 17:22:15 - [PeriodicallyPutsJob] The time is now 2017-11-06 17:22:15 and the wizard is feeling political
2017-11-06 17:22:15 - task periodically_puts_job<...> succeeded, took 0.0 seconds
```

More information: [periodic jobs on the wiki](https://github.com/robacarp/mosquito/wiki/Periodic-Jobs)

## Throttling Jobs

Jobs can be throttled to limit the number of messages that get executed within a given period of time.  For example, if 10 messages were enqueued for `ThrottledJob` at one time; 5 would be executed immediately, then pause for a minute, then execute the last 5.  

```crystal
class ThrottledJob < Mosquito::QueuedJob
  params message : String
  throttle limit: 5, period: 60

  def perform
    puts message
  end
end
```




## Connecting to Redis

Mosquito uses [Redis](https://redis.io/) to schedule jobs and store metadata about those jobs. Conventionally, redis connections are configured by an environment variable. To pass that configuration to Mosqito, 

```crystal
Mosquito.configure do |settings|
  settings.redis_url = ENV["REDIS_URL"]
end
```


## Contributing

Contributions are welcome. Please fork the repository, commit changes on a branch, and then open a pull request.

### Testing

This repository uses [minitest](https://github.com/ysbaddaden/minitest.cr) for testing. As a result, `crystal spec` doesn't do anything helpful. Do this instead:

```
make test
```

In lieu of `crystal spec` bells and whistles, Minitest provides a nice alternative to [running one test at a time instead of the whole suite](https://github.com/ysbaddaden/minitest.cr/pull/31).

## Contributors

- [robacarp](https://github.com/robacarp) creator, maintainer
