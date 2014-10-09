Mi Biblia de Rails
==================

Guia con los metodos, patrones y gemas mas usados en Rails.

## Gemas

## ActiveRecord

### Finders

```ruby
# Where
Post.where(author: 'admin')

# Find_by
Post.find_by(title: 'Rails 4')
Post.find_by("published_on < ?", 2.weeks.ago)

# Find_by with hash
post_params = { title: 'Rails 4', author: 'admin' }
Post.find_by(post_params)

# Find_or_*
Post.find_or_initialize_by(title: 'Rails 4')
Post.find_or_create_by(title: 'Rails 4')￼
```

### Updaters

```ruby
# This is the preferred way
@post.update(title: 'Rails 4', author: 'admin')

# These skip validations, builds a SQL statement and executes it directly in the database
@post.update_columns(title: 'Rails 4', author: 'admin')
```

### Scopes

```ruby
# Scopes should take a proc object
scope :sold, ->{ where(state: 'sold') }

# Defaults scopes should take a proc object or a block
default_scope ->{ where(status: 'available') }

# Eager-loaded scopes
# Use proc for resolve date in every query
scope :recent, ->{ where(published_at: 2.weeks.ago) }
scope :recent_red, ->{ recent.where(color: 'red') }
```

### Relations

```ruby
# None, returns ActiveRecord::Relationn and never hits the database
Post.none

# Not
Post.where.not(author: 'admin')

# Order
User.order(:name, created_at: :desc) # Default order is ASC

# Joins
Post.includes(:comments).where(comments: {name: 'foo' })
Post.includes(:comments).where('comments.name' => 'foo')
```

## ActiveModel::Model

Permite crear un modelo que no esta mapeado a la base de datos, puediendo disfrutar de las ventajas de ActiveRecord (validaciones, callbacks etc...) pero sin 
trabajar sobre una base de datos

```ruby
class SupportTicket
  include ActiveModel::Model

  attr_accessor :title, :description

  validates_presence_of :title
  validates_presence_of :description
end
```

### Solr
