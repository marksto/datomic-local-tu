# datomic-local-tu

Test utility for Datomic Local.

```clojure
org.clojars.marksto/datomic-local-tu {:mvn/version "1.0.2"}
```

## Rationale

Datomic Local provides a way to create clean environments useful for testing.
It does not provide a methodology for how to use it.
This library provides an opinionated way to use `:datomic-local` in your tests.

## Usage

The primary function exposed by the api is `test-env`.
This function returns a test environment.
A test environment will contain a Datomic client, created by Datomic Local,
that can be cleaned up by simply calling `.close`.
A test environment is typically used within a `with-open`.

```clojure
(require '[datomic-local-tu.core :as datomic-local-tu]
         '[datomic.client.api :as d])

(with-open [db-env (datomic-local-tu/test-env)]
  (let [_ (d/create-database (:client db-env) {:db-name "test"})
        conn (d/connect (:client db-env) {:db-name "test"})
        _ (d/transact conn {:tx-data [{:db/ident       ::name
                                       :db/valueType   :db.type/string
                                       :db/cardinality :db.cardinality/one}]})
        {:keys [tempids]} (d/transact conn {:tx-data [{:db/id "a"
                                                       ::name "hi"}]})]
    (d/pull (d/db conn)
            [::name]
            (get tempids "a"))))
=> #:example1{:name "hi"}
```

If you would prefer to manage the lifecycle, use `new-env` to generate a new,
random system and `cleanup-env!` to remove the associated resources.

Alternatively, you can use test-env with fixtures.
See `examples/example_fixture.clj` for the complete example.

```clojure
(def ^:dynamic *client* nil)

(defn client-fixture
  [f]
  (with-open [db-env (datomic-local-tu/test-env)]
    (binding [*client* (:client db-env)]
      (f))))

(use-fixtures :each client-fixture)
```

## Author

Kenny Williams @kennyjwilli
