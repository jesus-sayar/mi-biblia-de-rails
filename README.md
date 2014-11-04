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
```

### Joins

Rails tiene 2 formas de realizar joins. Uno está utilizando querys separadas para obtener los datos adicionales. Y la otra es utilizando una consulta (con LEFT OUTER JOIN) para obtener los datos adicionales.
Si usas #preload, significa que quieres queries separadas.
Si usas #eager_load usarás una unica query.
Si usas #includes, Rails decidirá por ti el camino a seguir.
Si usas #joins y le pasas el nombre de una asociación Rails realizará una query con INNER JOIN, si usas #joins y le pasas un string puedes realizar un OUTER JOIN.

```ruby
User.preload(:addresses)

#  SELECT "users".* FROM "users" 
#  SELECT "addresses".* FROM "addresses" WHERE "addresses"."user_id" IN (1, 2)

User.eager_load(:addresses)

#  SELECT
#  "users"."id" AS t0_r0, "users"."name" AS t0_r1, "users"."email" AS t0_r2, "users"."created_at" AS t0_r3, "users"."updated_at" AS t0_r4, 
#  "addresses"."id" AS t1_r0, "addresses"."user_id" AS t1_r1, "addresses"."country" AS t1_r2, "addresses"."street" AS t1_r3, "addresses"."postal_code" AS t1_r4, "addresses"."city" AS t1_r5, "addresses"."created_at" AS t1_r6, "addresses"."updated_at" AS t1_r7 
#  FROM "users" 
#  LEFT OUTER JOIN "addresses" ON "addresses"."user_id" = "users"."id"

Category.joins(:posts)

#   SELECT categories.* FROM categories INNER JOIN posts ON posts.category_id = categories.id

Client.joins('LEFT OUTER JOIN addresses ON addresses.client_id = clients.id')
#   SELECT clients.* FROM clients LEFT OUTER JOIN addresses ON addresses.client_id = clients.id

```

#### References
http://blog.arkency.com/2013/12/rails4-preloading/


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
