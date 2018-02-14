[![Gem Version](https://badge.fury.io/rb/isolator.svg)](https://badge.fury.io/rb/isolator)
[![Build Status](https://travis-ci.org/palkan/isolator.svg?branch=master)](https://travis-ci.org/palkan/isolator)

# Isolator

Detect non-atomic interactions within DB transactions.

Examples:

```ruby
# HTTP calls within transaction
User.transaction do
  user = User.new(user_params)
  user.save!
  # HTTP API call
  PaymentsService.charge!(user)
end

#=> raises Isolator::HTTPError

# background job
User.transaction do
  user.update!(confirmed_at: Time.now)
  UserMailer.successful_confirmation(user).deliver_later
end

#=> raises Isolator::BackgroundJobError
```

Isolator is supposed to be used in tests and on staging.

## Installation

Add this line to your application's Gemfile:

```ruby
group :development, :test do
  gem "isolator"
end
```

## Usage

Isolator is a plug-n-play tool, so, it begins to work right after required.

However, there are some potential caveats:

1) Isolator tries to automatically detect the environment and include only necessary adapters. Thus the order of loading gems matters: make sure that `isolator` is required in the end.

2) Isolator does not distinguish framework-level adapters. For example, `:active_job` spy doesn't take into account which AJ adapter you use; if you are using a safe one (e.g. `Que`) just disable the `:active_job` adapter to avoid false negatives (i.e. `Isolator.adatpers.active_job.disable!`).

3) Isolator tries to detect the `test` environment and slightly change it behaviour: first, it respect _transactional tests_; secondly, error raising is turned on by default (see [below](#configuration)).

### Configuration

```ruby
Isolator.configure do |config|
  # Specify a custom logger to log offenses
  config.logger = nil

  # Raise exception on offense
  config.raise_exceptions = false # true in test env

  # Send notifications to uniform_notifier
  config.send_notifications = false
end
```

Isolator relys on [uniform_notifier][] to send custom notifications.

**NOTE:** `uniform_notifier` should be installed separately (i.e. added to Gemfile).

### Adapters

Isolator has a bunch of built-in adapters:
- `:http` – built on top of [Sniffer][]
- `:active_job`
- `:sidekiq`

## Custom Adapters

## Contributing

Bug reports and pull requests are welcome on GitHub at https://github.com/palkan/isolator.

## License

The gem is available as open source under the terms of the [MIT License](http://opensource.org/licenses/MIT).

[Sniffer]: https://github.com/aderyabin/sniffer
[uniform_notifier]: https://github.com/flyerhzm/uniform_notifier