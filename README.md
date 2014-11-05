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

Para realizar un JOIN Rails proporcional el metodo "joins" que puede ser utilizado de multiples formas.

#### Usando Strings con fragmentos SQL

Puedes utilizar SQL puro para espedificar las clausulas del JOIN.

```ruby
Client.joins('LEFT OUTER JOIN addresses ON addresses.client_id = clients.id')

# Resultado 
SELECT clients.* FROM clients LEFT OUTER JOIN addresses ON addresses.client_id = clients.id
```

#### Usando Array/Hash de nombres de Asociaciones

ActiveRecord te deja utilizar los nombres de las asociaciones definidas en tu modelo como shorcuts para espedificar las clausulas del JOIN. Este metodo solo trabaja con INNER JOIN.

```ruby
Category.joins(:posts)

# Resultado 
SELECT categories.* FROM categories INNER JOIN posts ON posts.category_id = categories.id

Post.joins(:category, :comments)

# Resultado 
SELECT posts.* FROM posts INNER JOIN categories ON posts.category_id = categories.id INNER JOIN comments ON comments.post_id = posts.id

```

MAS AQUI http://guides.rubyonrails.org/active_record_querying.html#joining-tables

### Eager Loading Associations

Eager Loading es el mecanismo para cargar los registros de las asociaciones de un objeto con el menor numero de consultas.

#### N + 1 queries problem

Consideremos el siguiente codigo que busca 10 clientes y sus postcodes.

```ruby
clients = Client.limit(10)
 
clients.each do |client|
  puts client.address.postcode
end
```

El problema es el número de consultas ejecutadas. Este codigo ejecuta 1 consulta (para encontrar 10 clientes) + 10 (una por cada cliente para cargar su address) = 10 consultas en total.


#### Solución al N + 1 queries problem

ActiveRecord te deja especificar de forma avanzada todas las asociaciones que van a ser cargadas posteriormente. Esto es posible utilizando el metodo "#includes" sobre el modelo buscado. Con "#includes", ActiveRecord se asegura que todas las asociaciones espedificadas son cargadas previamente usando el mininmo numero de consultas.

La solución al codigo anterior utilizando "eager load" de la address seria así:

```ruby
clients = Client.includes(:address).limit(10)
 
clients.each do |client|
  puts client.address.postcode
end

# Resultado
SELECT * FROM clients LIMIT 10
SELECT addresses.* FROM addresses WHERE (addresses.client_id IN (1,2,3,4,5,6,7,8,9,10))
```

### Eager Loading Multiple Associations

ActiveRecord te permite realizar hacer "eager load" o "carga temprana" de cualquier numero de asociaciones de tu modelo usando el metodo "#includes" con un array/hash.

```ruby
# Recuperar todos los los posts y cargar todas las categorias y comentarios de cada post
Post.includes(:category, :comments)
```

### Especificar condiciones en Eager Loaded Associations

Aunque ActiveRecord te permite espedificar condiciones en el "eager loaded association" (asociaciones precargadas) de forma similar a como se hace con "#joins", el camino recomendado es usar #joins en su lugar.

Sin embargo, si tu debes hacer esto, tu puedes usar "#where" como se usa habitualmente.

```ruby
Post.includes(:comments).where(comments: { visible: true })

SELECT "posts"."id" AS t0_r0, ... "comments"."updated_at" AS t1_r5 FROM "posts" LEFT OUTER JOIN "comments" ON "comments"."post_id" = "posts"."id" WHERE (comments.visible = 1)
```

Cuando usas "#where" puede ser posible el uso de "#references"

```ruby
Post.includes(:comments).where("comments.visible = true").references(:comments)
```

### Metodos similares a #includes

El metodo #includes realizada la carga previa utilizando las consultas que ActiveRecord estime mejor, pero existe la posiblidad de usar otros dos metodos para indicar la estrategia de precarga.

Si usas #preload, significa que quieres queries separadas.
Si usas #eager_load usarás una unica query.
Si usas #includes, Rails decidirá por ti el camino a seguir.

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
