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

### Output
Upon running the command to start the worker, what you should see is output similar to the one below.

This particular example output (taken from the Foobara website linked [on this page](https://github.com/foobara/foobara-www/blob/main/src/Home/Connectors/Resque/ConnectorCodeOutput.ts)) comes from a running a different command called `IncrementAge`.

```sh
export default `$ QUEUE=* VERBOSE=1 rake resque:work
*** Starting worker demo:29545:general
*** got: (Job{general} | Foobara::CommandConnectors::ResqueConnector::CommandJob | [{"command_name" => "IncrementAge", "inputs" => {"capybara" => 1}}])
*** done: (Job{general} | Foobara::CommandConnectors::ResqueConnector::CommandJob | [{"command_name" => "IncrementAge", "inputs" => {"capybara" => 1}}])
*** got: (Job{general} | Foobara::CommandConnectors::ResqueConnector::CommandJob | [{"command_name" => "IncrementAge", "inputs" => {"capybara" => 2}}])
*** done: (Job{general} | Foobara::CommandConnectors::ResqueConnector::CommandJob | [{"command_name" => "IncrementAge", "inputs" => {"capybara" => 2}}])
*** got: (Job{general} | Foobara::CommandConnectors::ResqueConnector::CommandJob | [{"command_name" => "IncrementAge", "inputs" => {"capybara" => 2}}])
*** done: (Job{general} | Foobara::CommandConnectors::ResqueConnector::CommandJob | [{"command_name" => "IncrementAge", "inputs" => {"capybara" => 2}}])
*** ^C Exiting...
$ ./capybaras-cli FindAllCapybaras
{
  id: 1,
  name: "Fumiko",
  nickname: "foo",
  age: 101
},
{
  id: 2,
  name: "Barbara",
  nickname: "bar",
  age: 102
}`
```

## Contributing

Contributions in the form of Bug reports and pull requests are always welcome at [foobara/resque-connector](https://github.com/foobara/resque-connector/)

To work on an existing issue or submit a Pull Request:

1. Fork the repository and clone it onto your local machine
2. Run `bundle install` to install dependencies
3. Run `rake` to ensure that everything works as expected before making any changes
4. Implement your changes
5. Re-run `rake` to make sure that all tests and Rubocop pass
6. Commit your changes, push to Github and open a pull request for review

## License

This project is licensed under the MPL-2.0 license. Please see [LICENSE.txt](LICENSE.txt) for more information.
