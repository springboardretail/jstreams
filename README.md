# jstreams

[![Build](https://github.com/springboardretail/jstreams/actions/workflows/build.yml/badge.svg?branch=master)](https://github.com/springboardretail/jstreams/actions/workflows/build.yml)
[![Test Coverage](https://img.shields.io/codeclimate/coverage/jstotz/jstreams.svg)](https://codeclimate.com/github/jstotz/jstreams/test_coverage)
[![Maintainability](https://img.shields.io/codeclimate/maintainability/jstotz/jstreams.svg)](https://codeclimate.com/github/jstotz/jstreams/maintainability)
[![Docs](https://img.shields.io/badge/docs-yard-green.svg)](https://www.rubydoc.info/github/jstotz/jstreams/master)

A distributed streaming platform for Ruby built on top of Redis Streams.

Provides a multi-threaded publisher/subscriber.

## Project Status

This is alpha software and not suitable for production use.

## Features

- Load balancing among subscribers within a group
- Automatically message reassignment when consumers go away
- Multi-threaded subscribers
- Configurable message serialization

## Roadmap

- Configurable retry logic
- Replay a stream from a given point
- Wildcard subscriptions

## Installation

Add this line to your application's Gemfile:

```ruby
gem 'jstreams'
```

And then execute:

    $ bundle

Or install it yourself as:

    $ gem install jstreams

## Usage

### Publisher

```ruby
jstreams = Jstreams::Context.new

jstreams.publish(
  :users,
  event: 'user_created',
  user_id: 1,
  name: 'King Buzzo',
  email: 'buzzo@example.com'
)

jstreams.publish(:users, event: 'user_logged_in', user_id: 1)
```

### Subscriber

```ruby
jstreams = Jstreams::Context.new

jstreams.subscribe(
  :user_activity_logger,
  :users
) do |message, _stream, _subscriber|
  case message['event']
  when 'user_created'
    logger.info "User #{message['name']} created"
  when 'user_logged_in'
    logger.info "User #{message['id']} logged in"
  end
end

jstreams.subscribe(
  :send_welcome_email,
  :users
) do |message, _stream, _subscriber|
  send_user_welcome_email(message['id']) if message['event'] == 'user_created'
end

# Spawns subscriber threads and blocks
jstreams.run
```

### Replay

Starts a temporary copy of the given subscriber until messages have been replayed up to the checkpoint stored at the time replay is called.

```ruby
jstreams.replay(:user_activity_logger, from: message_id)
```

### Retries

By default subscribers will process messages indefinitely until successful.

```ruby
# TODO
```

### Serialization

```ruby
class Serializer
  MESSAGE_TYPES = {
    user_created: UserCreatedMessage, user_logged_in: UserLoggedInMessage
  }

  def serialize(type, message)
    message_class(type).serialize(message)
  end

  def deserialize(type, message)
    message_class(type).deserialize(message)
  end

  private

  def message_class(type)
    MESSAGE_TYPES.fetch(type) { raise "Unknown message type: #{type}" }
  end
end

jstreams = Jstreams::Context.new(serializer: Serializer)
```

## Development

After checking out the repo, run `bin/setup` to install dependencies. Then, run `rake spec` to run the tests. You can also run `bin/console` for an interactive prompt that will allow you to experiment.

To install this gem onto your local machine, run `bundle exec rake install`. To release a new version, update the version number in `version.rb`, and then run `bundle exec rake release`, which will create a git tag for the version, push git commits and tags, and push the `.gem` file to [rubygems.org](https://rubygems.org).

## Contributing

Bug reports and pull requests are welcome on GitHub at https://github.com/jstotz/jstreams. This project is intended to be a safe, welcoming space for collaboration, and contributors are expected to adhere to the [Contributor Covenant](http://contributor-covenant.org) code of conduct.

## License

The gem is available as open source under the terms of the [MIT License](https://opensource.org/licenses/MIT).

## Code of Conduct

Everyone interacting in the jstreams project’s codebases, issue trackers, chat rooms and mailing lists is expected to follow the [code of conduct](https://github.com/jstotz/jstreams/blob/master/CODE_OF_CONDUCT.md).
