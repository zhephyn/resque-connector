# Foobara::ResqueConnector

`foobara-resque-connector` connects Foobara commands to Resque for asynchronous use.

## Installation

Add `gem "foobara-resque-connector"` to your Gemfile or .gemspec file and `bundle install`.

Or even just `gem install foobara-resque-connector` if playing with it directly in the scripts.

## Usage

To set this up automatically in an existing Foobara project, run `foob g resque-connector`.

Otherwise, for the manual setup, save the following code to a file, such as `example.rb`.

```ruby
require "foobara/resque_connector" # require library

# Tell Resque where your Redis server lives.
Resque.redis = Redis.new(url: ENV.fetch("REDIS_URL"))

class DoSomething < Foobara::Command # defines Foobara command to run asynchronously
  inputs do 
    foo :integer
    bar :string
  end

  def execute
    "success! #{foo} #{bar}"
  end
end

connector = Foobara::CommandConnectors::ResqueConnector.new
connector.connect(DoSomething, queue: :general)
```

Queue the Resque job by running the command below:

```sh
REDIS_URL=redis://localhost:6379 ruby -r ./example -e 'DoSomethingAsync.run!(foo: 1, bar: "bar")'
```

Start the worker by running the following command:

```sh
REDIS_URL=redis://localhost:6379 QUEUE=general VERBOSE=1 rake resque:work
```

- `QUEUE=general` tells the worker which queue to take jobs from.

It must match the queue you passed to `connect`, or the worker won't see your jobs.

- To watch every queue, use `QUEUE=*`.

## Contributing

Contributions are welcome on Github at [foobara/resque-connector](https://github.com/foobara/resque-connector/)

To work on an existing issue or submit a Pull Request:

1. Fork the repository and clone it onto your local machine
2. Run `bundle install` to install dependencies
3. Run `rake` to ensure that everything works as expected before making any changes
4. Implement your changes
5. Re-run `rake` to make sure that all tests and Rubocop pass
6. Commit your changes, push to Github and open a pull request for review

## License

This project is licensed under the MPL-2.0 license. Please see [LICENSE.txt](LICENSE.txt) for more information.
