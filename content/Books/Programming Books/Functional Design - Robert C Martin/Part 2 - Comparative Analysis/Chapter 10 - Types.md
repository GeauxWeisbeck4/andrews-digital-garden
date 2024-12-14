---
id: 01JF1NDYGVCHSNHMJ5KFDHXTV9
modified: 2024-12-13T23:04:47-05:00
---
## 10

## Types

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9780138176518/files/graphics/10_unnum_types.jpg)

The preceding chapter may have left you somewhat distressed. Those things that I called objects were just hash maps and were completely untyped. Anybody could stick anything into them without any constraint. The salary in the `:pay-class` could hold a string instead of a number. The `:schedule` field could hold an integer instead of the appropriate keyword.

In short, these objects are not statically typed. The compiler does not check them. And therefore, _all hell could break loose_!

Many functional languages, as well as many OO languages, are statically typed in order to prevent that hell. Other languages, like Clojure, Python, and Ruby, depend upon other mechanisms to prevent that hell.

Those of us who practice TDD are not usually very concerned about that hell. Our tests generally ensure that the objects that we pass around are properly constructed. Still, in complex systems, where the totality of all the objects can end up being quite complex, there is a need for a more formal and complete way to ensure the integrity of our types than a dynamically typed language (and even most statically typed languages) can give us.

In Clojure, I use the `clojure.spec` library to achieve the goal of type integrity. The type specification for our payroll example looks like this:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch10_images.xhtml#f110-01)

```
(s/def ::id string?)
(s/def ::schedule #{:monthly :weekly :biweekly})
(s/def ::salaried-pay-class (s/tuple #(= % :salaried) pos1?))
(s/def ::hourly-pay-class (s/tuple #(= % :hourly) pos?))
(s/def ::commissioned-pay-class (s/tuple #(= % :commissioned)
                                         pos? pos?))
(s/def ::pay-class (s/or :salaried ::salaried-pay-class
                         :Hourly ::hourly-pay-class
                         :Commissioned ::commissioned-pay-class))
(s/def ::mail-disposition (s/tuple #(= % :mail) string? string?))
(s/def ::deposit-disposition (s/tuple #(= % :deposit)
                                      string? string?))
(s/def ::paymaster-disposition (s/tuple #(= % :paymaster)
                                        string?))
(s/def ::disposition (s/or :mail ::mail-disposition
                           :deposit ::deposit-disposition
                           :paymaster ::paymaster-disposition))
(s/def ::employee (s/keys :req-un [::id ::schedule
                                   ::pay-class ::disposition]))
(s/def ::employees (s/coll-of ::employee))

(s/def ::date string?)
(s/def ::time-card (s/tuple ::date pos?))
(s/def ::time-cards (s/map-of ::id (s/coll-of ::time-card)))

(s/def ::sales-receipt (s/tuple ::date pos?))
(s/def ::sales-receipts (s/map-of
                           ::id (s/coll-of ::sales-receipt)))

(s/def ::db (s/keys :req-un [::employees]
                    :opt-un [::time-cards ::sales-receipts]))

(s/def ::amount pos?)
(s/def ::name string?)
(s/def ::address string?)
(s/def ::mail-directive (s/and #(= (:type %) :mail)
                               (s/keys :req-un [::id
                                                ::name
                                                ::address
                                                ::amount])))

(s/def ::routing string?)
(s/def ::account string?)
(s/def ::deposit-directive (s/and #(= (:type %) :deposit)
                                  (s/keys :req-un [::id
                                                   ::routing
                                                   ::account
                                                   ::amount])))
(s/def ::paymaster string?)
(s/def ::paymaster-directive (s/and #(= (:type %) :paymaster)
                                    (s/keys :req-un [::id
                                                     ::paymaster
                                                     ::amount])))

(s/def ::paycheck-directive (s/or
                              :mail ::mail-directive
                              :deposit ::deposit-directive
                              :paymaster ::paymaster-directive))

(s/def ::paycheck-directives (s/coll-of ::paycheck-directive))
```

[1](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch10.xhtml#ch10fn1). `(pos? x)` returns true if `x` is a number greater than zero.

If this looks scary, it should. There’s a lot of detail in there. Keep in mind, however, that this is the level of detail that you would have to specify within the modules of a statically typed language in order to capture all the type constraints.

Understanding this type specification is not actually difficult. Look down toward the middle and find the definition of `::db`. This just says that the database is a hash map with a required `:employees` field and two optional fields for `:time-cards` and `:sales-receipts`.

If you look a bit higher in the specification, you’ll see that `::employees` is just a collection of `::employee`, `::sales-receipts` is a collection of `::sales-receipt`, and `::time-cards` is a collection of `::time-card`. Don’t let the double colons bother you; they are a namespace convention. You can read the Clojure docs later if you want to understand them. For now, just look at the keywords and ignore how many colons there are.

As we continue to work our way up, we see that an `::employee` is a hash map that is required to have the keys `:id`, `:schedule`, `:pay-class`, and `:disposition`. Keep exploring and you’ll find that the `:id` must be a string; the `:schedule` must be one of `:monthly`, `:weekly`, or `:biweekly`; and a `:salaried-pay-class` is a tuple containing `:salaried`, followed by a positive number.

The `s/or` statements might bother you a bit. The arguments come in pairs, and the first in each pair is just the name of that alternative. So, in the `::disposition` definition, `:mail` is just the name of the `::mail-disposition` alternative. Don’t worry anymore about this. It will become clear if you decide one day to read the `clojure.spec` docs.

So, given this elaborate type specification, how do we use it? I sometimes use it in my tests as follows:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch10_images.xhtml#f113-01)

```
(it "pays one salaried employee at end of month by mail"
  (let [employees [{:id "emp1"
                    :schedule :monthly
                    :pay-class [:salaried 5000]
                    :disposition [:mail "name" "home"]}]
        db {:employees employees}
        today (parse-date "Nov 30 2021")]
    (should (s/valid? ::db db))
    (let [paycheck-directives (payroll today db)]
      (should (s/valid? ::paycheck-directives 
                     paycheck-directives))
      (should= [{:type :mail
                 :id "emp1"
                 :name "name"
                 :address "home"
                 :amount 5000}]
               paycheck-directives))))
```

Look for the calls to `s/valid?`, which is a function that returns true if the data matches the spec. Look carefully and you’ll see that I’m checking the `::db` spec on the way in and the `::paycheck-directives` spec on the way out. This is pretty secure. If my tests have high coverage, and they all check the specs for the inputs and outputs of the functions they call, then violations of type ought to be extremely rare.

I have, upon occasion, also used Clojure’s `:pre` and `:post` features to run the specs on critical data before and after the main processing functions of my applications.

Here, for example, is the main processing step of the `spacewar`[2](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch10.xhtml#ch10fn2a) game I wrote some years ago:

[2](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch10.xhtml#ch10fn2). [https://github.com/unclebob/spacewar](https://github.com/unclebob/spacewar)

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch10_images.xhtml#f114-01)

```
(defn update-world [ms world]
  ;{:pre [(valid-world? world)]
  ; :post [(valid-world? %)]}
  (->> world
       (game-won ms)
       (game-over ms)
       (ship/update-ship ms)
       (shots/update-shots ms)
       (explosions/update-explosions ms)
       (clouds/update-clouds ms)
       (klingons/update-klingons ms)
       (bases/update-bases ms)
       (romulans/update-romulans ms)
       (view-frame/update-messages ms)
       (add-messages)))
```

The `:pre` and `:post` statements are commented out,[3](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch10.xhtml#ch10fn3a) but they are ready to be reasserted should I suspect some kind of terrible type corruption.

[3](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch10.xhtml#ch10fn3). I don’t much care for commented-out code. I’d remove these lines as the project matured.

### Conclusion

There is a lot of wailing and gnashing of teeth over the static versus dynamic typing issue. Each side yells at the other without listening to what either side has to say. I think both sides have valid points. Dynamic typing makes code easier to write. Static typing makes code a lot safer, easier to understand, and much more internally consistent. It seems to me that a library like `clojure.spec` strikes a great balance. It gives you the ability to have as much or as little type checking as you need. It allows you to specify when types _are_ checked and when they are _not_. What’s more, it allows you to specify dynamic constraints that no static type system _can_ check. So, for my money, libraries like this give you better than the best of both worlds.