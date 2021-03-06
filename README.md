# Async::IO

Async::IO provides builds on [async] and provides asynchronous wrappers for `IO`, `Socket`, and related classes.

[async]: https://github.com/socketry/async

[![Build Status](https://secure.travis-ci.org/socketry/async-io.svg)](http://travis-ci.org/socketry/async-io)
[![Code Climate](https://codeclimate.com/github/socketry/async-io.svg)](https://codeclimate.com/github/socketry/async-io)
[![Coverage Status](https://coveralls.io/repos/socketry/async-io/badge.svg)](https://coveralls.io/r/socketry/async-io)

## Installation

Add this line to your application's Gemfile:

```ruby
gem 'async-io'
```

And then execute:

	$ bundle

Or install it yourself as:

	$ gem install async-socket

## Usage

Basic echo server (from `spec/async/io/echo_spec.rb`):

```ruby
require 'async/io'

def echo_server(endpoint)
	Async::Reactor.run do |task|
		# This is a synchronous block within the current task:
		endpoint.accept do |client|
			# This is an asynchronous block within the current reactor:
			data = client.read(512)
			
			# This produces out-of-order responses.
			task.sleep(rand * 0.01)
			
			client.write(data.reverse)
		end
	end
end

def echo_client(endpoint, data)
	Async::Reactor.run do |task|
		endpoint.connect do |peer|
			result = peer.write(data)
			
			message = peer.read(512)
			
			puts "Sent #{data}, got response: #{message}"
		end
	end
end

Async::Reactor.run do
	endpoint = Async::IO::Endpoint.tcp('0.0.0.0', 9000)
	
	server = echo_server(endpoint)
	
	5.times.collect do |i|
		echo_client(endpoint, "Hello World #{i}")
	end.each(&:wait)
	
	server.stop
end
```

## Contributing

1. Fork it
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create new Pull Request

## See Also

- [async](https://github.com/socketry/async) — Asynchronous event-driven reactor.
- [async-dns](https://github.com/socketry/async-dns) — Asynchronous DNS resolver and server.
- [async-rspec](https://github.com/socketry/async-rspec) — Shared contexts for running async specs.
- [rubydns](https://github.com/ioquatix/rubydns) — A easy to use Ruby DNS server.

## License

Released under the MIT license.

Copyright, 2017, by [Samuel G. D. Williams](http://www.codeotaku.com/samuel-williams).

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in
all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
THE SOFTWARE.
