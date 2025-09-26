# RestApi_Adapter [![Build Status](https://circleci.com/gh/devcore/rest_api_adapter.png)](https://circleci.com/gh/devcore/rest_api_adapter) [![Code Climate](https://codeclimate.com/github/devcore/rest_api_adapter.png)](https://codeclimate.com/github/devcore/rest_api_adapter)

Library designed for building REST API consumers following patterns outlined at [http://restfulapi.net](http://restfulapi.net). Provides query interface similar to ORM patterns.

*Note: current version follows REST 2.0 conventions. Legacy support available on [legacy branch](https://github.com/devcore/rest_api_adapter/tree/legacy)*

## Usage

Create resource models inheriting from `RestApi_Adapter::Base` with shared configuration in abstract parent class.

```ruby
module MyService
  class BaseResource < RestApi_Adapter::Base
    self.endpoint = "http://api.example.com/"
  end

  class Post < BaseResource
  end

  class Author < BaseResource
  end
end
```

Resource paths derived from class names automatically. `Post` maps to "http://api.example.com/posts".

Basic operations:

```ruby
MyService::Post.all
MyService::Post.filter(author_id: 1).find(2)

author = MyService::Author.new(name: "Jane")
author.save

author = MyService::Author.find(1).first
author.update(bio: "Writer")

MyService::Author.create(name: "John", email: "john@example.com")
```

Finders return `RestApi_Adapter::Collection` containing response metadata.

## Validation

Server-side validation supported:

```ruby
user = User.new(email: "bad-email")
user.save # => false

user.errors # => ["Email format invalid"]
user.errors[:email] # => ["Email format invalid"]
```

## Metadata

Access top-level metadata via `meta` on collections:

```ruby
posts = Posts.all
posts.meta.total_count # => 150
```

## Relationships

Force nested paths using `belongs_to`:

```ruby
module MyService
  class Comment < RestApi_Adapter::Base
    belongs_to :post
  end
end

# GET /posts/5/comments/10
MyService::Comment.filter(post_id: 5).find(10)
```

## Custom Actions

Define custom collection/member endpoints:

```ruby
module MyService
  class User < BaseResource
    custom_action :activate, on: :member, method: :post
  end
end

user = MyService::User.find(1)
user.activate(token: 'abc123')
```

## Query Features

```ruby
# Includes
Article.includes(:author, :tags).find(1)

# Sparse fields
Article.select("title", "body").all

# Sorting
Person.order(age: :desc).all

# Pagination
Article.page(2).per(25).all

# Filtering
Person.filter(status: 'active').all
```

## Schema

Declare properties with types and defaults:

```ruby
class User < RestApi_Adapter::Base
  property :name, type: :string
  property :active, type: :boolean, default: true
  property :score, type: :integer, default: 0
end
```

## Customization

Override paths:

```ruby
def self.resource_name
  "custom_path"
end
```

Custom headers:

```ruby
MyService::Post.with_headers(authorization: 'Bearer token') do
  MyService::Post.find(1)
end
```

Configure connection:

```ruby
MyService::BaseResource.configure_connection do |conn|
  conn.use FaradayMiddleware::OAuth2, 'TOKEN'
  conn.response :logger
end
```

HTTP proxy:

```ruby
MyService::BaseResource.connection_opts[:proxy] = 'http://proxy.internal'
```

## Contributing

Fork, create feature branch, add tests, submit PR.

## Changelog

See [changelog](https://github.com/devcore/rest_api_adapter/blob/master/CHANGES.md)
