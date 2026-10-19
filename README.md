# Foobara::RedisCrudDriver

`foobara-redis-crud-driver` is a Redis CRUD driver for Foobara.

## Installation

Add `gem "foobara-redis-crud-driver"` to your Gemfile or .gemspec file and `bundle install`. 

Or even just `gem install foobara-redis-crud-driver` if just playing with it directly in the scripts.

## Usage

Save the code below in a file such as `widget_example.rb`. This sets up the Redis CRUD driver, defines a `Widget` entity plus two commands: one to create a widget and one to find it.

```ruby
require "foobara/redis_crud_driver" # require the driver

driver = Foobara::RedisCrudDriver.new(ENV.fetch("REDIS_URL"), key_prefix: "my_app")
Foobara::Persistence.default_crud_driver = driver # set the driver as Foobara's default CRUD driver for saving entities

class Widget < Foobara::Entity
  attributes do
    id :integer
    name :string, :required
  end

  primary_key :id
end

class CreateWidget < Foobara::Command
  inputs Widget.attributes_for_create
  result Widget

  def execute
    Widget.create(inputs)
  end
end

class FindWidget < Foobara::Command
  inputs do
    id Widget.primary_key_type, :required
  end
  result Widget

  def execute
    Widget.load(id)
  end
end

widget = CreateWidget.run!(name: "Example")
puts FindWidget.run!(id: widget.id).name # => "Example"
```

Running the command below in your project folder will print `Example` as the output signifying that the widget is now stored in Redis. 

```sh
REDIS_URL=redis://localhost:6379 ruby widget_example.rb
```

### Things to know

- **Use commands, or open a transaction**. Foobara requires an open transaction for successful entity CRUD operations. The commands in the example above open one for you. If you call `Widget.create` or `Widget.load` directly, outside a command, you need to have opened a Foobara transaction yourself.
- **Key prefix** `key_prefix:` keeps this driver's keys under your own label. If several apps share one Redis, give each its own. The older name `prefix:` still works for compatibility but is deprecated, so use `key_prefix:`.

### Choosing how to connect

The first argument to `Foobara::RedisCrudDriver.new` can be any of these:

- **A Redis URL**, like in the above example
- **An existing `Redis` client** you've already created.
- **A credentials hash**, which is anything `Redis.new` accepts.
You can also leave the connection out. The driver then reads `ENV["REDIS_URL"]` itself. If that variable isn't set either, it raises `Foobara::RedisCrudDriver::NoRedisUrlError`.

## Contributing

Bug reports and pull requests are always welcome at [github.com/foobara/redis-crud-driver](https://github.com/foobara/redis-crud-driver).

To work on an existing issue or submit a Pull Request:

1. Fork the repository and clone it to your local machine
2. Run `bundle install` to install dependencies
3. Start a Redis server and run `rake` to ensure that everything works as expected before making any changes
4. Implement your changes
5. Restart the Redis server and re-run `rake` to make sure that all tests and Rubocop pass
6. Commit your changes, push to GitHub and open a Pull Request for review

## License

This project is licensed under the MPL-2.0 license. Please see [LICENSE.txt](LICENSE.txt) for more information.
