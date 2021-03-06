# naan

Naan provides CRUD functions for working with [Korma](http://sqlkorma.com/) entities.

What is naan?  It isn't korma.  It doesn't replace korma.  It doesn't wrap around korma.  It's just bread... er... CRUD for Korma.  It is a nice-to-have that goes well with korma.  Korma is excellent and does many things very well.  However, I do find that for create, read, update, and destroy operations, I really want to DRY-up my code.  Nann provides these four functions, and a few more helpers.  Try naan with korma, its satisfying... tasty even.

## Assumptions

- You are using korma, of course.
- You do a lot of simple CRUD operations and are tired of writing the same code.
- Your tables have an integer key column ('id' by default) and a string candidate key column ('name' by default).  Each of these are optional and configurable.
- Naan provides little more than simple CRUD helpers, so it assumes that you will write more complex SQL using korma. Naan shouldn't ever get in your way.
- You can customize your entity definitions when you want to override naan defaults for a particular table.

## Usage

### Include korma and naan

```clojure
(ns some.namespace
  (:require
    [korma.core :as korma]
    [naan.core :as naan]))
```

### CRUD with korma and naan


Use defdb and defentity as normal.

```clojure
(korma/defentity users
  (korma/entity-fields :id :first :last)
  (korma/pk :id)
  (korma/database db))
```

You could create an instance of the entity using korma.

```clojure
(korma/insert users
  (korma/values {:first "Tasty", :last "SQL"}))
```

That's not bad.  But naan is a bit shorter.

```clojure
(naan/create users {:first "Tasty", :last "SQL"})
```

Or, supply multiple entities using a collection:

```clojure
(naan/create users [{:first "Tasty", :last "SQL"} {:first "Crunchy" :last "Naan"}])
```

Meh.  That isn't much different.  Now read that user using a korma select.

```clojure
(first
  (korma/select users
    (korma/where {:id [= 1]})
    (korma/limit 1)))
```

And using naan read.

```clojure
(naan/read users 1)
```

You can also find entities by other fields.

```clojure
(naan/read users {:first "Crunchy", :last "Naan"})
```

That's a very straightforward helper function.  Let's see updating with korma update.


```clojure
(korma/update users
  (korma/set-fields {:last "Korma"})
  (korma/where {:id [= 1]}))
```

Naan is shorter.

```clojure
(naan/update users {:last "Korma"} 1)
```

Finally destroying in korma.

```clojure
(korma/delete users
  (korma/where {:id [= 1]}))
```

And again with naan.

```clojure
(naan/destroy users 1)
```

CRUD with naan makes korma more satisfying.

### String keys

It is pretty common to use an 'id' column on our tables.  It seems like we also commonly give things names, and CRUD should just work.

```clojure
(korma/defentity cats
  (korma/entity-fields :name :breed :color :gender)
  (korma/pk :id)
  (korma/database db))

(naan/create cats {:name "Crookshanks", :breed "Tabby", :color "Orange", :gender "M"})
(naan/read cats "Crookshanks")
```

We should also be able to specify an alternate string key.

```clojure
(korma/defentity cats
  (korma/entity-fields :nickname :breed :color :gender)
  (naan/set-string-key :nickname)
  (korma/pk :id)
  (korma/database db))

(naan/create cats {:nickname "Crookshanks", :breed "Tabby", :color "Orange", :gender "M"})
(naan/read cats "Crookshanks")
```

Or maybe we just want to override the default.

```clojure
(korma/defentity cats
  (korma/entity-fields :name :owner :breed :color :gender)
  (korma/pk :id)
  (korma/database db))

(naan/create cats {:name "Crookshanks", :owner "Granger", :breed "Tabby", :color "Orange", :gender "M"})
(binding [*string-key* :owner] (naan/read cats "Granger"))
```

### Using Maps

Maps can also be used instead of keys.

```clojure
(korma/defentity cats
  (korma/entity-fields :name :breed :color :gender)
  (korma/pk :id)
  (korma/database db))

(naan/create cats {:name "Crookshanks", :breed "Tabby", :color "Orange", :gender "M"})
(naan/read cats {:color "Orange"})
(naan/update cats {:color "Tawny"} {:color "Orange"})
(naan/destroy cats {:breed "Tabby"})
```

### With timestamps

Naan can update created_at and update_at timestamps, as appropriate, on CRUD operations.  It requires adding a bit of boiler-plate to your entity definitions, but it pays off when timestamps (a la Rails) just work.

```clojure
(require '[naan.korma-helpers :as helpers]

(korma/defentity cats
  (helpers/attributes :name :breed :color :gender :created_at :updated_at)
  (helpers/entity-fields-from-attributes)
  (korma/pk :id)
  (korma/database db))
```

## Copyright

Naan is Copyright © 2013 Stephen Sloan, and is funded by [Rafter.com](http://www.rafter.com "Rafter.com").  It is MIT licensed.

![Rafter Logo](http://rafter-logos.s3.amazonaws.com/rafter_github_logo.png "Rafter")
