---
id: 01JF1NH0DRAD3GN8MN8FYP4QZZ
modified: 2024-12-13T23:06:27-05:00
---
## 12

## Solid

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9780138176518/files/graphics/29_unnum_solid.jpg)

I wrote about the SOLID principles over two decades ago in the context of OO design. Because of that context, many have come to associate those principles with OO and regard them as anathema to functional programming. This is unfortunate because the SOLID principles are general principles of software design that are not specific to any particular programming style. In this chapter, I will endeavor to explain how the SOLID principles apply to functional programming.

The following chapters are summaries, not complete descriptions, of the principles. For those of you who are interested in more detail, I recommend the following sources.

- _Agile Software Development: Principles, Patterns, and Practices_.[1](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch12.xhtml#ch12fn1a)
    
    [1](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch12.xhtml#ch12fn1). Robert C. Martin (Pearson, 2002).
    
- _Clean Architecture_.[2](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch12.xhtml#ch12fn2a)
    
    [2](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch12.xhtml#ch12fn2). Robert C. Martin (Pearson, 2017).
    
- [Cleancoder.com](http://cleancoder.com/). Check out the blog posts and articles. There are lots and lots of things to learn on this Web site about principles and more.
    
- [Cleancoders.com](http://cleancoders.com/). This Web site has videos that explain each principle in great detail and with compelling examples.
    

### The Single Responsibility Principle (SRP)

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9780138176518/files/graphics/12_unnum_srp.jpg)

The _SRP_ is a simple statement about focusing our modules on the sources that cause them to change. Those sources are, of course, _people_. It is people who request changes to software, and therefore it is people to whom our modules are responsible.

These people can be separated into groups called _roles_ or _actors_. An actor is a person, or a group of people, who require the same things from the system. The kinds of changes they request will be consistent with each other. On the other hand, different actors have different needs. The changes one actor requests will affect the system in very different ways from the changes requested by other actors. Those disparate changes may even be at cross purposes to each other.

When a module is responsible to more than one actor, the changes requested by those competing actors can interfere with each other. This interference often leads to the design smell of _fragility_; causing the system to break in unexpected ways when simple changes are made.

Nothing can be quite so terrifying to managers and customers than systems that suddenly misbehave in startling ways after simple feature changes are made. If this repeats too often, the only conclusion they can come to is that the developers have lost control of the system and don’t know what they are doing.

A violation of the SRP can be as simple as mixing GUI formatting and business rule code together in the same module. Or it can be as complex as using stored procedures in the database to implement business rules.

Here’s a simple example of a nasty SRP violation written in Clojure. First, let’s look at the tests because they tell the story:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch12_images.xhtml#f127-01)

```
(describe "Order Entry System"
  (context "Parsing Customers"
    (it "parses a valid customer"
      (should=
        {:id "1234567"
         :name "customer name"
         :address "customer address"
         :credit-limit 50000}
        (parse-customer
          ["Customer-id: 1234567"
           "Name: customer name"
           "Address: customer address"
           "Credit Limit: 50000"])))

    (it "parses invalid customer"
      (should= :invalid
               (parse-customer
                 ["Customer-id: X"
                  "Name: customer name"
                  "Address: customer address"
                  "Credit Limit: 50000"]))
      (should= :invalid
               (parse-customer
                 ["Customer-id: 1234567"
                  "Name: "
                  "Address: customer address"
                  "Credit Limit: 50000"]))
      (should= :invalid
               (parse-customer
                 ["Customer-id: 1234567"
                  "Name: customer name"
                  "Address: "
                  "Credit Limit: 50000"]))
      (should= :invalid
               (parse-customer
                 ["Customer-id: 1234567"
                  "Name: customer name"
                  "Address: customer address"
                  "Credit Limit: invalid"])))
    (it "makes sure credit limit is <= 50000"
      (should= :invalid
               (parse-customer
                 ["Customer-id: 1234567"
                  "Name: customer name"
                  "Address: customer address"
                  "Credit Limit: 50001"])))))
```

The first test tells us that we are parsing some text input into a customer record. That record has four fields: `id`, `name`, `address`, and `credit-limit`. The next four tests tell us about syntax errors such as missing or malformed input.

The last test is the interesting one. It tests a business rule. Testing a business rule as part of parsing the input is a clear SRP violation. The parsing code can safely validate syntax errors, but it should avoid all _semantic_ checks because those checks are in the domain of a different actor. The actor who specifies the input format is not the same as the actor who specifies the largest allowable credit limit.[3](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch12.xhtml#ch12fn3a)

[3](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch12.xhtml#ch12fn3). This is true even when the two actors are the same person. In that case, that person is playing two different roles.

The code that passes these tests exacerbates the problem:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch12_images.xhtml#f129-01)

```
(defn validate-customer
  [{:keys [id name address credit-limit] :as customer}]
  (if (or (nil? id)
          (nil? name)
          (nil? address)
          (nil? credit-limit))
    :invalid
    (let [credit-limit (Integer/parseInt credit-limit)]
      (if (> credit-limit 50000)
        :invalid
        (assoc customer :credit-limit credit-limit)))))

(defn parse-customer [lines]

  (let [[_ id] (re-matches #"^Customer-id: (\d{7})$"
                           (nth lines 0))
        [_ name] (re-matches #"^Name: (.+)$" (nth lines 1))
        [_ address] (re-matches #"^Address: (.+)$" (nth lines 2))
        [_ credit-limit] (re-matches #"^Credit Limit: (\d+)$"
                                     (nth lines 3))]
    (validate-customer
      {:id id
       :name name
       :address address
       :credit-limit credit-limit})))
```

Look at how the `validate-customer` function mixes the syntax checks with the semantic business rule that limits the credit limit to 50,000. That semantic check belongs in an entirely different module, not tangled in with all those syntax checks.

Worse, consider a programmer who conscientiously uses `clojure/spec` to dynamically define the type of `customer`:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch12_images.xhtml#f130-01)

```
(s/def ::id (s/and
              string?
              #(re-matches #"\d+" %)))
(s/def ::name string?)
(s/def ::address string?)
(s/def ::credit-limit (s/and int? #(<= % 50000)))
(s/def ::customer (s/keys :req-un [::id ::name
                                   ::address ::credit-limit]))
```

This specification properly constrains the customer data structure to be syntactically correct; but it also imposes the semantic business rule constraint that the credit limit must not be greater than 50,000.

Why am I concerned about mixing the credit limit constraint with the syntax of the data structure? It is because I expect the syntax of the data structure and the credit limit constraint to be specified by different actors. And I expect those different actors will request changes at different times and for different reasons. I don’t want a change to the syntax to inadvertently break a business rule.

Of course, this begs the question: Where do semantic validations belong? The answer to that is semantic validations belong in the modules responsible to the actors who are likely to change them. If, for example, there is a business rule that says that credit limits must not exceed 50,000, then the enforcement code should go in the module that handles all the other credit limit processing.

_Gather together the things that change for the_

_same reasons, and at the same times._

_Separate those things that change for different_

_reasons or at different times._

### The Open-Closed Principle (OCP)

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9780138176518/files/graphics/13_unnum_ocp.jpg)

The _OCP_ was first stated by Bertrand Meyer in his classic 1988 book, _Object-Oriented Software Construction_.[4](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch12.xhtml#ch12fn4a) To paraphrase, it says that software modules should be open for extension but closed for modification. This means that you want to design your modules such that extending or changing their behavior does not require you to modify their code.

[4.](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch12.xhtml#ch12fn4) Pearson, 1988.

This may sound oxymoronic, but it’s actually something that we do all the time. Consider, for example, the `copy` program in C:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch12_images.xhtml#f131-01)

```
void copy() {
  int c;
  while ((c = getchar()) != EOF)
    putchar(c);
}
```

This program copies characters from `stdin` to `stdout`. I can add new devices to the operating system anytime I like. For example, I could add an optical character recognition (OCR) and a text-to-speech synthesizer to the system. This program would still operate without complaint and would happily copy characters from the OCR to the voice synthesizer without needing to be modified or even recompiled.

This is a very powerful idea that allows us to separate high-level policy from low-level detail and keep the high-level policy immune from changes to the low-level detail. However, it requires that the high-level policy access the low-level detail through an abstraction layer.

In OO programs, we typically create that abstraction layer through polymorphic interfaces. In statically typed languages like Java, C#, and C++, those interfaces are classes[5](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch12.xhtml#ch12fn5a) with abstract methods. High-level policies are given access through those interfaces to the low-level details that implement, or inherit from, those interfaces.

[5](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch12.xhtml#ch12fn5). The keyword `interface` in Java and C# defines classes where every method is abstract.

In dynamically typed OO languages like Python and Ruby, these interfaces are duck types. _Duck types_ have no particular syntax within the language. They are simply sets of function signatures called by the high-level policies and implemented by the low-level details. The dynamic type system determines the polymorphic dispatch at runtime by matching those signatures.

Some functional languages, like F# and Scala, sit on top of an OO foundation and thus can take advantage of the polymorphic interfaces of that foundation. But functional languages have long had another mechanism by which the abstraction layer for the OCP can be created: functions.

#### Functions

Consider this simple Clojure program:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch12.xhtml#f133-01a)

```
(defn copy [read write]
  (let [c (read)]
    (if (= c :eof)
      nil
      (recur read (write c)))))
```

This is essentially the same program as the `copy` program written in C, except that the functions to read and write have been passed in as arguments.[6](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch12.xhtml#ch12fn6a) Nevertheless, the abstraction layer for the OCP is intact.

[6.](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch12.xhtml#ch12fn6) Functions that are passed as arguments, or returned as values from functions, are sometimes called _higher-order functions_.

By the way, I tested this program using the following tests. I think you’ll find this interesting.

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch12_images.xhtml#f133-01)

```
(def str-in (atom nil))
(def str-out (atom nil))

(defn str-read []
  (let [c (first @str-in)]
    (if (nil? c)
      :eof
      (do
        (swap! str-in rest)
        c))))

(defn str-write [c]
  (swap! str-out str c)
  str-write)

(describe "copy"
  (it "can read and write using str-read and str-write"
    (reset! str-in "abcedf")
    (reset! str-out "")
    (copy str-read str-write)
    (should= "abcdef" @str-out)))
```

I used the `atom`s because I/O is a side effect and is therefore not purely functional. After all, when you read from an input or write to an output, you are mutating their states. Thus, the low-level I/O functions are not purely functional and use Software Transactional Memory to manage the mutation of state.

#### Objects with Vtables

For those of you who are pining for OO, you can pass an “object” into `copy` using the following technique:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch12.xhtml#f134-01a)

```
(defn copy [device]
  (let [c ((:getchar device))]
    (if (= c :eof)
      nil
      (do
        ((:putchar device) c)
        (recur device)))))
```

The test simply loads the device map with the functions:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch12_images.xhtml#f134-01)

```
(it "can read and write using str-read and str-write"
    (reset! str-in "abcedf")
    (reset! str-out "")
    (copy {:getchar str-read :putchar str-write})
    (should= "abcdef" @str-out))
```

C++ programmers will recognize that the `device` argument is just a vtable—which is the polymorphism mechanism in C++. In any case, it should be obvious that you can define many different devices for the `copy` program to use. You can extend the behavior of `copy` without having to modify it.

#### Multi-methods

Still another variation on this theme is the use of multi-methods. Many languages, functional or otherwise, support multi-methods in one way or another. _Multi-methods_ are another form of duck typing, because they create a loose grouping of methods that are dynamically dispatched based on their function signature and the “type”[7](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch12.xhtml#ch12fn7a) of the arguments.

[7.](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch12.xhtml#ch12fn7) I used quotes here because the “type” of the arguments is not necessarily associated with their specific data types. Indeed, that “type” can be a completely different concept.

In Clojure, we use the time-honored approach of a _dispatching function_ to specify that “type”:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch12_images.xhtml#f135-01)

```
(defmulti getchar (fn [device] (:device-type device)))
(defmulti putchar (fn [device c] (:device-type device)))
```

Here we see `getchar` and `putchar` declared as multi-methods. Each has a dispatching function that takes the same arguments that `getchar` and `putchar` will be called with. We can change the `copy` program to call those multi-methods:

```
(defn copy [device]
  (let [c (getchar device)]
    (if (= c :eof)
      nil
      (do
        (putchar device c)
        (recur device)))))
```

The test for this new copy function is below. Notice that the test `device` is no longer a vtable containing pointers to functions. Instead, it now contains the input and output `atom`s, and also a `:device-type`. It is that `:device-type` that the multi-methods will be dispatching on.

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch12_images.xhtml#f135-02)

```
(it "can read and write using multi-method"
  (let [device {:device-type :test-device
                :input (atom "abcdef")
                :output (atom nil)}]
    (copy device)
    (should= "abcdef" @(:output device))))
```

All that remains are the implementations of the multi-methods. They should not be too surprising.

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch12_images.xhtml#f136-01)

```
(defmethod getchar :test-device [device]
  (let [input (:input device)
        c (first @input)]
    (if (nil? c)
      :eof
      (do
        (swap! input rest)
        c))))

(defmethod putchar :test-device [device c]
  (let [output (:output device)]
    (swap! output str c)))
```

These are the implementations that will be dispatched when the `:device-type` is `:test-device`. It should be clear that many other such implementation methods could be created for various different devices. Those new devices will extend the `copy` program without forcing any modification.

#### Independent Deployability

One of the benefits we expect to get from the OCP is the ability to compile high-level policies and low-level details in separate modules and to deploy them independently. In Java and C#, this would mean compiling them down into separate `jar` or `dll` files that can be dynamically loaded. In C++, we would compile the modules and place the binaries into dynamically loadable shared libraries.

The Clojure solutions shown above do not achieve that goal. The high-level policy and the low-level detail cannot be dynamically loaded from two separate `jar` files.

This is much less of an issue than it would be in Java or C# because “loading” a Clojure program almost always[8](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch12.xhtml#ch12fn8a) involves compiling it. Thus, while the high-level policies and low-level details may not be dynamically loaded from `jar` files, they are dynamically compiled and loaded from _source_ files. Therefore, most of the benefits of independently deployable `jar` files are preserved.

[8](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch12.xhtml#ch12fn8). Clojure allows for precompilation in some cases.

However, if you absolutely must have total and complete independent deployability, there is another option. You can use Clojure’s protocols and records:

```
(defprotocol device
  (getchar [_])
  (putchar [_ c]))
```

The protocol will become a Java `interface` that can be independently compiled into a `jar` file for dynamic loading. The implementation of the protocol (shown below) can likewise be independently compiled and loaded:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch12_images.xhtml#f137-01)

```
(defrecord str-device [in-atom out-atom]
  device
  (getchar [_]
    (let [c (first @in-atom)]
      (if (nil? c)
        :eof
        (do
          (swap! in-atom rest)
          c))))

  (putchar [_ c]
    (swap! out-atom str c)))

(describe "copy"
  (it "can read and write using str-read and str-write"
    (let [device (->str-device (atom "abcdef") (atom nil))]
      (copy device)
      (should= "abcdef" @(:out-atom device)))))
```

Notice the `->str-device` function in the test. That’s essentially the Java constructor of the `str-device` class that implements the `device` protocol. Notice also that I loaded the `atom`s into the device as in the previous example.

Indeed, I did not change the `copy` program to get this example to work. The `copy` program is exactly as it was in the multi-method example. Now that’s the OCP at work!

If the protocol/record mechanism of Clojure feels like OO, that’s because it is OO. The JVM is an OO foundation, and Clojure fits very nicely upon that foundation.

### The Liskov Substitution Principle (LSP)

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9780138176518/files/graphics/14_unnum_lsp.jpg)

Any language that supports the OCP must also support the LSP. The two principles are linked because every violation of the LSP is a latent violation of the OCP.

The _LSP_ was first described by Barbara Liskov in 1988,[9](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch12.xhtml#ch12fn9a) providing a more or less formal definition of a subtype. In essence, she said that a subtype must be substitutable for its base type in any program that uses the base type.

[9](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch12.xhtml#ch12fn9). Coincidentally, that’s the same year that Bertrand Meyer published the OCP.

To clarify that, let us say that we have some program `pay` that uses a type `employee`:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch12_images.xhtml#f139-01)

```
(defn pay [employee pay-date]
  (let [is-payday? (:is-payday employee)
        calc-pay (:calc-pay employee)
        send-paycheck (:send-paycheck employee)]
    (when (is-payday? pay-date)
      (let [paycheck (calc-pay)]
        (send-paycheck paycheck)))))
```

Notice that I’m using the vtable approach to create the type. Notice also that the data within the type is completely hidden from the `pay` function. All the `pay` function can see is the methods within the `employee` type. How much more OO can you get?

Here’s the test code that uses this type. Notice that the `make-test-employee` function makes an object that uses _duck typing_ to conform to the `employee` type:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch12_images.xhtml#f139-02)

```
(defn test-is-payday [employee-data pay-date]
  true)

(defn test-calc-pay [employee-data]
  (:pay employee-data))

(defn test-send-paycheck [employee-data paycheck]
  (format "Send %d to: %s at: %s"
          paycheck
          (:name employee-data)
          (:address employee-data)))
(defn make-test-employee [name address pay]
  (let [employee-data {:name name
                       :address address
                       :pay pay}

        employee {:employee-data employee-data
                  :is-payday (partial test-is-payday
                                      employee-data)
                  :calc-pay (partial test-calc-pay employee-data)
                  :send-paycheck (partial test-send-paycheck
                                          employee-data)}]

    employee))

(describe "Payroll"
  (it "pays a salaried employee"
    (should= "Send 100 to: name at: address"
             (pay (make-test-employee "name" "address" 100)
                  :now))))
```

Notice the `make-test-employee` function uses the pointer to implementation (PIMPL)[10](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch12.xhtml#ch12fn10a) pattern to hide the data in the `:employee-data` field and expose only the methods. Finally, notice that all the polymorphic methods are given the `employee-data` as their first arguments. Oh, just so OO! And yet entirely functional.

[10](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch12.xhtml#ch12fn10). Holding all the data behind a single field to help keep it private. See [https://cpppatterns.com/patterns/pimpl.html](https://cpppatterns.com/patterns/pimpl.html).

It should be clear that I could create many different kinds of employee objects and pass them to the `pay` function without modifying the `pay` function at all. This is the OCP.

However, to achieve that I must be very careful to make sure that every employee object I create conforms to the expectations of the `pay` function. If one of those methods does something that `pay` doesn’t expect, then `pay` will malfunction.

For example, this test fails:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch12_images.xhtml#f141-01)

```
      (it "does not pay an employee whose payday is not today"
        (should-be-nil
          (pay (make-later-employee "name" "address" 100)
               :now)))
```

It fails because `make-later-employee` does not conform to the `pay` function’s expectations for the :`is-payday` method. As you can see below, it returns `:tomorrow` instead of `false`:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch12_images.xhtml#f141-02)

```
(defn make-later-employee [name address pay]
  (let [employee (make-test-employee name address pay)
        is-payday? (partial (fn [_ _] :tomorrow)
                            (:employee-data employee))]
    (assoc employee :is-payday is-payday?)))
```

This is an LSP violation.

Now imagine you were the author of the `pay` function, and you were tasked with debugging why certain employees were getting paychecks at the wrong times. You find that many employee objects are using the `:tomorrow` convention instead of returning a boolean as they should. What do you do?[11](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch12.xhtml#ch12fn11a)

[11](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch12.xhtml#ch12fn11). Of course, a statically typed language would solve that particular issue. So would a well-timed call to `s/valid?`, given appropriate specs. But that’s not the case we are investigating at the moment.

You _could_ fix all those employees. Or you could add an extra condition to the `pay` function:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch12_images.xhtml#f141-03)

```
(defn pay [employee pay-date]
  (let [is-payday? (:is-payday employee)
        calc-pay (:calc-pay employee)
        send-paycheck (:send-paycheck employee)]
    (when (= true (is-payday? pay-date))
      (let [paycheck (calc-pay)]
        (send-paycheck paycheck)))))
```

Yeah, that’s pretty ugly.[12](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch12.xhtml#ch12fn12a) It’s also an OCP violation because we’ve modified high-level policy due to the misbehavior of a low-level detail.

[12](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch12.xhtml#ch12fn12). Think long and hard about why that is ugly and why many programmers would be tempted to delete the = true, thus re-exposing the bug.

#### The ISA Rule

The OO literature often uses the term _ISA_ (pronounced, and meaning, “is a”) to describe subtypes. To describe the above situation in those terms we would say that the `test-employee` ISA `employee`, and the `later-employee` ISA `employee`. This usage can be confusing.

First, the `later-employee` is not an `employee` because it does not conform to the expectations of the `pay` function; and it is the `pay` function, and all the other functions that operate on `employee`s, that define what the `employee` type is.

But second, and perhaps more important, the term _ISA_ can be deeply misleading. The ancient and venerable square/rectangle conundrum is often used to make this point.

Let us say that we have an object that describes a rectangle. In Clojure, it might look like this:

```
(defn make-rect [h w]
  {:h h :w w})
```

A simple test of this rectangle object might look like this:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch12_images.xhtml#f142-01)

```
(it "calculates proper area after change in size"
  (should= 12 (-> (make-rect 1 1) (set-h 3) (set-w 4) area)))
```

To make this work we’ll need the `set-h`, `set-w`, and `area` functions as follows:

```
(defn set-h [rect h]
  (assoc rect :h h))

(defn set-w [rect w]
  (assoc rect :w w))

(defn area [rect]
  (* (:h rect) (:w rect)))
```

Nothing here should be surprising. The rectangle object is not mutable. The `set-h` and `set-w` functions simply create new rectangles with the changed parameters.

So let’s flesh this out a bit and create a small system that uses our rectangle. Here are the tests:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch12_images.xhtml#f143-01)

```
(describe "Rectangle"
  (it "calculates proper area and perimeter"
    (should= 25 (area (make-rect 5 5)))
    (should= 18 (perimeter (make-rect 4 5)))
    (should= 12 (-> (make-rect 1 1) (set-h 3) (set-w 4) area)))

  (it "minimally increases area"
    (should= 15 (-> (make-rect 3 4) minimally-increase-area area))
    (should= 24 (-> (make-rect 5 4) minimally-increase-area area))
    (should= 20 (-> (make-rect 4 4) minimally-increase-area area))))
```

And here are the functions that pass those tests:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch12_images.xhtml#f143-02)

```
(defn perimeter [rect]
  (let [{:keys [h w]}13 rect]
    (* 2 (+ h w))))

(defn minimally-increase-area [rect]
  (let [{:keys [h w]} rect]
    (cond
      (>= h w) (make-rect (inc h) w)
      (> w h) (make-rect h (inc w))
      :else :tilt)))
```

[13](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch12.xhtml#ch12fn13). This _destructures_ the map into the named components. In this case, it is equivalent to `(let [h (:h rect) w (:w rect)]`…

Again, there’s nothing very surprising about this. Perhaps you are confused by the `minimally-increase-area` function. This function simply increases the area of the rectangle by the smallest integral amount possible.[14](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch12.xhtml#ch12fn14a)

[14](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch12.xhtml#ch12fn14). Presuming all the lengths and widths are integers.

So now let’s imagine that this system has been in operation for years and has been very successful. But lately the customers of this system have been asking for squares. How do we add squares to our system?

If we apply the ISA rule, we might decide that a square is a rectangle, and therefore, we should make the functions that accept rectangles also accept squares. In Java, we might accomplish this by deriving the class `Square` from the class `Rectangle`. In Clojure, we can do this by simply creating rectangles with equal sides:

```
(defn make-square [side]
  (make-rect side side))
```

This should bother us slightly because the size of the `square` object is the same as the size of the `rectangle` object. Objects of type `square` ought to be smaller since they don’t need both the height and the width. But memory is cheap, and we want to keep things simple, right?

The question is, will all our tests still pass? They should, of course, because our squares are really just rectangles (ah, that’s just the ISA rule!).

These tests pass just fine:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch12_images.xhtml#f145-01)

```
(should= 36 (area (make-square 6)))
(should= 20 (perimeter (make-square 5)))
```

So does this one, but it’s bothersome because somewhere in there, “squareness” got lost:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch12_images.xhtml#f145-02)

```
(should= 12 (-> (make-square 1) (set-h 3) (set-w 4) area))
```

The functions `set-h` and `set-w` do not return a `square` when passed a `square`. That’s a bit strange; but in some bizarre way it actually makes sense. I mean, if you set the height of a `square` without changing the width, it’s not going to be a `square` anymore, right?

If you feel a little itching at the back of your brain right now, you should probably pay attention to it.

Anyway, what about our `minimally-increase-area` test? Does it pass?

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch12_images.xhtml#f145-03)

```
(should= 30 (-> (make-square 5) minimally-increase-area area))
```

Yes, that passes too. And of course, it should since the function simply increases the height or width as necessary.

So it looks like we’re done, and this worked just great!

#### Nope!

Our customer calls us up a few days later, and he’s not very happy. He’s been trying to minimally increase the area of his squares, and it’s just not working.

“When I increase the area of a 5-by-5 square,” he bleats, “I get a rectangle back with an area of 30. I need to get a _square_ back with an area of 36!”

Uh-oh. Looks like we guessed wrong. This is an LSP violation. We created a subtype that does not conform to the expectations of the functions that use the base type. The expectation of `minimally-increase-area` is that height and width can be modified independently. According to our customer, that’s not true for a `square`.

So, what should we do?

We could add a `:type` field to the objects and have the constructors put either `:square` or `:rectangle` into the field, respectively. And of course, then we’d have to put an `if` statement into the `minimally-increase-area` function. We’d also have to change `set-h` and `set-w` to change the type to `:rectangle`. And those changes violate the OCP, because every violation of the LSP is a latent violation of the OCP.

I’ll leave other solutions as an exercise. You might try using multi-methods. You might try using protocols and records. You might try using vtables. Or you might just keep the two types absolutely separate and never pass a `square` into a function that takes a `rectangle`.

#### The Representative Rule

I prefer this last option. That’s because I don’t much care for the ISA rule. You see, while it is _geometrically_ true that a square is a rectangle, none of the objects in my code were actual rectangles or squares. My code had objects that _represented_ squares and rectangles, but they were _neither_ squares _nor_ rectangles. And here’s the thing about representatives:

_The representatives of things do not share the_

_relationships of the things they represent._

Just because a square is a rectangle in geometry, it does not mean that a `square` object in code is a `rectangle` object in code. That relationship is not shared because objects of type `square` do not behave the way objects of type `rectangle` behave.

When you see two objects in the real world that are obviously connected by the phrase “is a,” you may be tempted to create a subtype relationship in your code. Be careful with that. You may just run afoul of the representative rule and violate the LSP.

### The Interface Segregation Principle (ISP)

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9780138176518/files/graphics/15_unnum_isp.jpg)

The name of this principle derives from its origins in statically typed OO languages. The example I usually use to describe the ISP works quite well for such languages as Java, C#, and C++, because those languages depend upon declared interfaces. In dynamically typed languages like Ruby, Python, JavaScript, and Clojure, those examples don’t work particularly well, because in those languages, interfaces are undeclared and are already segregated by duck typing.

For example, consider the following Java interface:

```
interface AtmInteractor {
  void requestAccount();
  void requestAmount();
  void requestPin();
}
```

Here we see three methods bound together in the `AtmInteractor` interface. Any user of this interface therefore depends upon all three methods, even if that user only calls one of those methods. Thus, that user depends upon more than it needs. If the signature of one of those methods changes, or if another method is added to that interface, then that user will have to be recompiled and redeployed, making the design unnecessarily fragile.

We solve this weakness in statically typed OO languages by segregating the interfaces as follows:

```
interface AccountInteractor {
  void requestAccount();
}

interface AmountInteractor {
  void requestAmount();
}

interface PinInteractor {
  void requestPin();
}
```

Then each user can depend only upon the methods that it needs to call while the implementation can multiply implement those interfaces:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch12_images.xhtml#f148-01)

```
public class AtmInteractor implements AccountInteractor,
                                      AmountInteractor,
                                      PinInteractor {
  void requestAccount() {…};
  void requestAmount() {…};
  void requestPin() {…};
}
```

Perhaps the UML diagram in [Figure 12.1](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch12.xhtml#ch12fig01) will make this clearer. By segregating the interfaces, the three users depend only on the methods that they need; and yet those methods can be implemented by a single class.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9780138176518/files/graphics/c12f001.jpg)

**Figure 12.1.** Segregated interfaces

In Clojure, we could use one of our duck typing techniques to address this problem:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch12_images.xhtml#f149-01)

```
(defmulti request-account :interactor)
(defmulti request-amount :interactor)
(defmulti request-pin :interactor)
```

Those three multi-methods are not bound together under a single declaration. Indeed, they do not even need to be kept together in the same source file. They could instead be declared in modules that are specific to their function. Thus, if the signature of one changed, or if a new multi-method were added, there would be no impact upon the users of the multi-methods that were not changed. If they were precompiled,[15](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch12.xhtml#ch12fn15a) they would not require recompilation.

[15](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch12.xhtml#ch12fn15). Clojure allows modules to be precompiled for faster loading.

This means that in dynamically typed languages, like Clojure, it is easier to avoid depending on things you don’t need. But that doesn’t mean that the principle doesn’t apply.

#### Don’t Depend on Things You Don’t Need

Back to the name. The word _Interface_ in _Interface Segregation Principle_ is not tied solely to the interface classes in Java, C#, and C++. Rather, it applies to the generic meaning of the word. The “interface” of a module is simply the list of all the access points within that module.

Java and C# (and, by strong convention, C++) are class-based languages in which there is a strong coupling between classes and source files. Java in particular demands that each source file be named after the sole public class declared within that source file. This automatically sets up the conditions that the ISP is trying to avoid. Groups of methods are coupled together into a single module that users will depend upon, even if they don’t depend upon every one of those methods. Thus, unless the designer is careful, those users will depend upon things they don’t need.

Dynamically typed languages like Ruby, Python, and Clojure do not have this class-to-module constraint. You can declare anything you like within any source file you like. You can write the entire application in a single source file if you like![16](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch12.xhtml#ch12fn16a) Therefore, it is even easier in those languages to set up the conditions that will cause users of a module to depend upon things they don’t need.

[16](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch12.xhtml#ch12fn16). Not recommended. ;-)

This is not a situation that is specific to functional languages. It is also not a situation from which functional languages are immune. Designers can easily pollute the interfaces of their modules with all kinds of access points that the majority of their users don’t need.

#### Why?

Why do we care about depending on modules that have more than we need? Why should it bother us if our module only uses one of the ten functions in another module?

In statically typed languages the cost can be severe because a change to one of the functions we don’t use can force our module to be recompiled and redeployed. If our module is just one of many modules in a binary component (like a `jar` file), then that entire component will need to be redeployed. Those are couplings that every serious designer should be careful about.

In dynamically typed languages, the cost is reduced but is not zero. In Clojure, for example, there is a strict requirement[17](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch12.xhtml#ch12fn17a) that the source code dependencies between modules must be acyclic. The more functions that a module contains, the more outgoing and incoming source code dependencies impinge upon that module and thus the greater the probability that it will participate in a cycle.

[17](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch12.xhtml#ch12fn17). We’ll encounter this in [Chapter 17](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch17.xhtml#ch17), [Wa-Tor](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch17.xhtml#ch17).

But possibly the best reason for caring about these dependencies is that a module structure that limits extraneous dependencies is _cogent_. It is an indication that intelligent human beings have cared enough to separate the concerns and lower the coupling. The readers of your code will thank you for that care.

#### Conclusion

The real meaning of the ISP is:

_Gather together the things that are used together._

_Separate those things that are used separately._

_Don’t depend on things you don’t need._

### The Dependency Inversion Principle (DIP)

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9780138176518/files/graphics/16_unnum_dip.jpg)

Of the SOLID principles, one could say that the OCP is the moral heart, the SRP is the organizing force, while the LSP and the ISP are caution signs surrounding the potholes created by carelessness. That leaves the DIP, which is the underlying mechanism behind all the others. In almost every case when we find a principle violation, the solution involves the inversion of one or more critical dependencies.

In decades long past, software was constructed with a completely constrained and parallel dependency structure. Source code dependencies paralleled runtime dependencies. The structure looked like [Figure 12.2](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch12.xhtml#ch12fig02).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9780138176518/files/graphics/c12f002.jpg)

**Figure 12.2.** The ancient parallel dependency structure

The dashed arrows are runtime dependencies. They show that high-level modules call mid-level modules, which call low-level modules. The solid arrows are source code dependencies. They show that each source code module depends upon the modules it calls. Those source code dependencies were statements like `#include`, `import`, `require`, and `using` that mentioned the name of the downstream source file.

In those ancient days of yore, those two kinds of dependencies were always[18](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch12.xhtml#ch12fn18a) parallel to each other. If module `X` had a runtime dependency on module `Y`, it also had a source code dependency on module `Y`.

[18](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch12.xhtml#ch12fn18). Well, not quite always. In the late ‘50s and early ‘60s, Herculean efforts were expended by operating system engineers to invert a few, very strategic dependencies in order to create the abstraction of device independence. They had no tool other than explicit pointers to functions, so they were very, very careful.

This meant that high-level policy was inextricably dependent upon low-level detail. Think hard about the implications of that statement.

But in the late ‘60s, Ole-Johan Dahl and Kristen Nygaard moved a data structure[19](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch12.xhtml#ch12fn19a) in the ALGOL compiler from the stack to the heap and discovered OO.[20](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch12.xhtml#ch12fn20a) And with that discovery came the ability for programmers to invert dependencies easily and safely.

[19](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch12.xhtml#ch12fn19). The data structure was the stack frame of function calls. The language they created was Simula 67.

[20](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch12.xhtml#ch12fn20). The history of the invention of Simula is fascinating. It is briefly described in the 1972 book _Structured Programming_ by Edsger W. Dijkstra, Ole-Johan Dahl, and C. A. R. Hoare (Academic Press), and in much more detail in the paper “The Development of the Simula Languages” by Dahl and Nygaard ([https://hannemyr.com/cache/knojd_acm78.pdf](https://hannemyr.com/cache/knojd_acm78.pdf)).

It took another 25 years before OO languages started to move into the mainstream. But since then, virtually all programmers have been able to effortlessly break that parallel dependence. They do it as shown in [Figure 12.3](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch12.xhtml#ch12fig03).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9780138176518/files/graphics/c12f003.jpg)

**Figure 12.3.** Inverting the dependency by inserting an interface

`HL1` has a runtime dependency on `F()` within `ML1`; but `HL1` has no source code dependency, either direct or transitive, upon `ML1`. Instead, they both depend upon the interface `I`.[21](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch12.xhtml#ch12fn21a)

[21](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch12.xhtml#ch12fn21). In dynamically typed languages, the interface `I` would not exist as a source code module. Rather, it would be a duck type that `HL1` and `ML1` would conform to.

This ability to take any source code dependency and invert it provides us with an immense amount of power. We can easily and safely arrange the source code dependencies of our software to ensure that high-level modules _do not_ depend upon low-level modules.

This allows us to create structures like that shown in [Figure 12.4](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch12.xhtml#ch12fig04).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9780138176518/files/graphics/c12f004.jpg)

**Figure 12.4.** Plug-in structure

Here we see the high-level business rules have runtime dependencies upon the user interface (UI) and the database but have no source code dependencies on those modules. This application of the DIP means that the UI and database are _plug-ins_ to the business rules and could easily be replaced with different implementations without affecting the business rules, thereby conforming to the OCP.

Of course, what’s really going on is that the UI and the database are implementing interfaces contained within the business rules. The business rules operate upon those interfaces, allowing the flow of control to go outward toward the UI and database while keeping the source code dependencies inverted inward toward the business rules (see [Figure 12.5](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch12.xhtml#ch12fig05)).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9780138176518/files/graphics/c12f005.jpg)

**Figure 12.5.** The interfaces within the business rules allow plug-ins.

Notice that all the dependencies point toward abstractions. This leads us to one way to describe the DIP:

_Where possible, point all source code dependencies at abstractions._

#### A Blast from the Past

But enough theory. Let’s see this at work. I’m going to borrow a nostalgic example from my friend and mentor, Martin Fowler. He presented this _Video Store_[22](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch12.xhtml#ch12fn22a) example in the first edition of his wonderful book, _Refactoring_.[23](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch12.xhtml#ch12fn23a) Of course, I’m going to use Clojure instead of Java.

[22](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch12.xhtml#ch12fn22). Video killed the radio store and the Internet killed the video store. Yes, boys and girls, there was a time when we would go to the video store to rent videotapes and DVDs.

[23](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch12.xhtml#ch12fn23). Addison-Wesley, 1999.

Here are the tests:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch12_images.xhtml#f155-01)

```
(describe "Video Store"
  (with customer (make-customer "Fred"))

  (it "makes statement for a single new release"
    (should= (str "Rental Record for Fred\n"
                  "\tThe Cell\t9.0\n"
                  "You owed 9.0\n"
                  "You earned 2 frequent renter points\n")
             (make-statement
               (make-rental-order
                 @customer
                 [(make-rental
                    (make-movie "The Cell" :new-release)
                    3)]))))

  (it "makes statement for two new releases"
    (should= (str "Rental Record for Fred\n"
                  "\tThe Cell\t9.0\n"
                  "\tThe Tigger Movie\t9.0\n"
                  "You owed 18.0\n"
                  "You earned 4 frequent renter points\n")
             (make-statement
               (make-rental-order
                 @customer
                 [(make-rental
                    (make-movie "The Cell" :new-release)
                    3)
                  (make-rental
                    (make-movie "The Tigger Movie" :new-release)
                    3)]))))

  (it "makes statement for one childrens movie"
    (should= (str "Rental Record for Fred\n"
                  "\tThe Tigger Movie\t1.5\n"
                  "You owed 1.5\n"
                  "You earned 1 frequent renter points\n")
             (make-statement
               (make-rental-order
                 @customer
                 [(make-rental
                    (make-movie "The Tigger Movie" :childrens)
                    3)]))))

  (it "makes statement for several regular movies"
    (should= (str "Rental Record for Fred\n"
                  "\tPlan 9 from Outer Space\t2.0\n"
                  "\t8 1/2\t2.0\n"
                  "\tEraserhead\t3.5\n"
                  "You owed 7.5\n"
                  "You earned 3 frequent renter points\n")
             (make-statement
               (make-rental-order
                 @customer
                 [(make-rental
                    (make-movie "Plan 9 from Outer Space" :regular)
                    1)
                  (make-rental
                    (make-movie "8 1/2", :regular)
                    2)
                  (make-rental
                    (make-movie "Eraserhead" :regular)
                    3)])))))
```

From these tests, you should be able to determine what this application does. Customers rent videos for a certain number of days. The price and the reward points are apparently calculated based upon the type of the video and the number of days they are rented. There seem to be three types of videos: `:regular`, `:new-release`, and `:childrens`.

Here is the code that passes these tests:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch12_images.xhtml#f157-02)

```
(defn make-customer [name]
  {:name name})

(defn make-movie [title type]
  {:title title
   :type type})

(defn make-rental [movie days]
  {:movie movie
   :days days})

(defn make-rental-order [customer rentals]
  {:customer customer
   :rentals rentals})

(defn determine-amount [rental]
  (let [{:keys [movie days]} rental
        type (:type movie)]
    (condp = type
      :regular
      (if (> days 2)
        (+ 2.0 (* (- days 2) 1.5))
        2.0)

      :new-release
      (* 3.0 days)

      :childrens
      (if (> days 3)
        (+ 1.5 (* (- days 3) 1.5))
        1.5))))

(defn determine-points [rental]
  (let [{:keys [movie days]} rental
          type (:type movie)]
    (if (and (= type :new-release)
             (> days 1))
      2
      1)))

(defn make-detail [rental]
  (let [title (:title (:movie rental))
        price (determine-amount rental)]
    (format "\t%s\t%.1f" title price)))

(defn make-details [rentals]
  (map make-detail rentals))

(defn make-footer [rentals]
  (let [owed (reduce + (map determine-amount rentals))
        points (reduce + (map determine-points rentals))]
    (format
      "\nYou owed %.1f\nYou earned %d frequent renter points\n"
      owed points)))

(defn make-statement [rental-order]
  (let [{:keys [name]} (:customer rental-order)
        {:keys [rentals]} rental-order
        header (format "Rental Record for %s\n" name)
        details (string/join "\n" (make-details rentals))
        footer (make-footer rentals)]
    (str header details footer)))
```

If you read the first edition of _Refactoring_, this should look pretty familiar. In essence, we have a simple report generator that calculates and formats a statement for a rental order.

The very first thing you should have noticed is the horrific SRP violation in the tests. Those tests couple the business rules with the construction and formatting of the statement. If someone from marketing decides to make even a trivial change to the statement format, all the tests will fail.

Consider, for example, the effects of changing the statement to begin with the words “Rental Statement for” instead of “Rental Record for.”

This SRP violation makes the tests very fragile. To fix this we need to separate the tests that specify the format of the report from the tests that specify the business rules.

To do this I’m going to split the tests into three different modules: one for testing the calculations, another for the formatting, and the last for integration.

Here is the `statement-calculator` test. From now on, I’ll include all the `ns`[24](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch12.xhtml#ch12fn24a) statements so that you can see the names of the modules and their source code dependencies.

[24](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch12.xhtml#ch12fn24). `ns` stands for namespace. These statements generally appear at the start of every Clojure module and define the module’s name and its dependencies.

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch12_images.xhtml#f159-01)

```
(ns video-store.statement-calculator-spec
  (:require [speclj.core :refer :all]
            [video-store.statement-calculator :refer :all]))
(declare customer)

(describe "Rental Statement Calculation"
  (with customer (make-customer "Fred"))

  (it "makes statement for a single new release"
    (should= {:customer-name "Fred"
              :movies [{:title "The Cell"
                        :price 9.0}]
              :owed 9.0
              :points 2}
             (make-statement-data
               (make-rental-order
                 @customer
                 [(make-rental
                    (make-movie "The Cell" :new-release)
                    3)]))))

  (it "makes statement for two new releases"
    (should= {:customer-name "Fred",
              :movies [{:title "The Cell", :price 9.0}
                       {:title "The Tigger Movie", :price 9.0}],
              :owed 18.0,
              :points 4}
             (make-statement-data
               (make-rental-order
                 @customer
                 [(make-rental
                    (make-movie "The Cell" :new-release)
                    3)
                  (make-rental
                    (make-movie "The Tigger Movie" :new-release)
                    3)]))))

  (it "makes statement for one childrens movie"
    (should= {:customer-name "Fred",
              :movies [{:title "The Tigger Movie", :price 1.5}],
              :owed 1.5,
              :points 1}
             (make-statement-data
               (make-rental-order
                 @customer
                 [(make-rental
                    (make-movie "The Tigger Movie" :childrens)
                    3)]))))

  (it "makes statement for several regular movies"
    (should= {:customer-name "Fred",
              :movies [{:title "Plan 9 from Outer Space",
                        :price 2.0}
                       {:title "8 1/2", :price 2.0}
                       {:title "Eraserhead", :price 3.5}],
              :owed 7.5,
              :points 3}
             (make-statement-data
               (make-rental-order
                 @customer
                 [(make-rental
                    (make-movie "Plan 9 from Outer Space"
                                :regular)
                    1)
                  (make-rental
                    (make-movie "8 1/2", :regular)
                    2)
                  (make-rental
                    (make-movie "Eraserhead" :regular)
                    3)])))))
```

What we’ve done here is replace the formatted rental statement with a data structure that contains all the data that goes into the statement. This allows us to separate the formatting from the calculation, as shown in the `statement-calculator` implementation:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch12_images.xhtml#f161-02a)

```
(ns video-store.statement-calculator)

(defn make-customer [name]
  {:name name})

(defn make-movie [title type]
  {:title title
   :type type})

(defn make-rental [movie days]
  {:movie movie
   :days days})

(defn make-rental-order [customer rentals]
  {:customer customer
   :rentals rentals})

(defn determine-amount [rental]
  (let [{:keys [movie days]} rental
        type (:type movie)]
    (condp = type
      :regular
      (if (> days 2)
        (+ 2.0 (* (- days 2) 1.5))
        2.0)

      :new-release
      (* 3.0 days)

      :childrens
      (if (> days 3)
        (+ 1.5 (* (- days 3) 1.5))
        1.5))))

(defn determine-points [rental]
  (let [{:keys [movie days]} rental
        type (:type movie)]
    (if (and (= type :new-release)
             (> days 1))
      2
      1)))

(defn make-statement-data [rental-order]
  (let [{:keys [name]} (:customer rental-order)
        {:keys [rentals]} rental-order]
    {:customer-name name
     :movies (for [rental rentals]
               {:title (:title (:movie rental))
                :price (determine-amount rental)})
     :owed (reduce + (map determine-amount rentals))
     :points (reduce + (map determine-points rentals))}))
```

This is a bit simpler than before and is nicely encapsulated. Notice the `ns` statement shows that this module has no source code dependencies. Everything in the module is about the calculation of the data that goes into the statement. However, there is nothing here that hints at the formatting of the statement.

The formatting test is quite simple:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch12_images.xhtml#f163-01)

```
(ns video-store.statement-formatter-spec
  (:require [speclj.core :refer :all]
            [video-store.statement-formatter :refer :all]))

(describe "Rental Statement Format"
  (it "Formats a rental statement"
    (should= (str "Rental Record for CUSTOMER\n"
                  "\tMOVIE\t9.9\n"
                  "You owed 100.0\n"
                  "You earned 99 frequent renter points\n")
             (format-rental-statement
               {:customer-name "CUSTOMER"
                :movies [{:title "MOVIE"
                          :price 9.9}]
                :owed 100.0
                :points 99}))))
```

This should be self-explanatory. We’re just making sure that we can format the data produced by the `statement-calculator` module. The implementation is also very simple:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch12_images.xhtml#f164-01)

```
(ns video-store.statement-formatter)

(defn format-rental-statement [statement-data]
  (let [customer-name (:customer-name statement-data)
        movies (:movies statement-data)
        owed (:owed statement-data)
        points (:points statement-data)]
    (str
      (format "Rental Record for %s\n" customer-name)
      (apply str
             (for [movie movies]
               (format "\t%s\t%.1f\n"
                       (:title movie)
                       (:price movie))))
      (format "You owed %.1f\n" owed)
      (format "You earned %d frequent renter points\n" points))))
```

Again, we have a nicely encapsulated module with no source code dependencies.

To make sure that both of these modules work together as they should, I added a simple integration test:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch12_images.xhtml#f164-02)

```
(ns video-store.integration-specs
  (:require [speclj.core :refer :all]
            [video-store.statement-formatter :refer :all]
            [video-store.statement-calculator :refer :all]))

(describe "Integration Tests"
  (it "formats a statement for several regular movies"
    (should= (str "Rental Record for Fred\n"
                  "\tPlan 9 from Outer Space\t2.0\n"
                  "\t8 1/2\t2.0\n"
                  "\tEraserhead\t3.5\n"
                  "You owed 7.5\n"
                  "You earned 3 frequent renter points\n")
             (format-rental-statement
               (make-statement-data
                 (make-rental-order
                   (make-customer "Fred")
                   [(make-rental
                      (make-movie
                        "Plan 9 from Outer Space" :regular)
                      1)
                    (make-rental
                      (make-movie "8 1/2", :regular)
                      2)
                    (make-rental
                      (make-movie "Eraserhead" :regular)
                      3)]))))))
```

This is much better from an SRP point of view. If the marketing folks make trivial changes to the format of the report, only the formatting and integration tests will break. None of the calculation tests will break. That might not seem like a big win in a toy example like this. But in a real-world application where the tests would number in the thousands, this is a very big win indeed.

We are also protected from business rule changes. If the finance people decide they need to change the way prices are calculated, the formatting test will be immune, and only the calculation and integration tests will be affected.

#### A DIP Violation

While all this winning was going on, did you happen to notice the DIP violation? You might have missed it because it’s not in the production code. It’s in the integration test.

Look at the `ns` statement. Do you see those two lines that mention the `statement-formatter` and the `statement-calculator`? Those lines create source code dependencies on the concrete implementations of those modules. That’s a high-level policy depending on a concrete low-level detail. That’s a definitional DIP violation.

Perhaps this puzzles you. How can a test be a high-level policy? Aren’t tests as low level as you can get? Aren’t they the ultimate details?

Yes, that’s true. But integration tests in particular are stand-ins for high-level policy. Look at that integration test again. It does precisely what the high-level policy of the application would have to do. It calls `make-statement-data` and passes the result to `format-rental-statement`. And since both of those functions are concrete implementations, our high-level production code will have the same DIP violation as our integration test.

Do we always pay attention to the DIP in our tests? It is always wise to be aware. It may not always be wise to force compliance. Some tests are best left coupled to low-level implementations. However, if you want your test suites to be robust and flexible and if you don’t want a hundred tests to break when you change one small thing in the production code, then keeping an eye on the coupling between your tests and the production code is a good idea.[25](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch12.xhtml#ch12fn25a)

[25](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch12.xhtml#ch12fn25). I spend a lot of time on this topic in my book _Clean Craftsmanship_ (Addison-Wesley, 2021).

But perhaps you are still not convinced. So let’s add a new feature. Sometimes we want the statement to be displayed on a text terminal, and sometimes we want it on a browser. So we need text and HTML versions of `format-rental-statement`.

Let’s also add one more new feature. Some of our stores are offering a “buy two, get one free” policy. So, if you rent three videos, you will only be charged for the two most expensive ones.

If we were implementing this in an OO language, we would likely be tempted to create two new abstract classes or interfaces. The `StatementFormatter` abstraction would have a `format-rental-statement` method that would be implemented in both the `TextFormatter` and `HTMLFormatter` implementations. Likewise, the `StatementPolicy` abstraction would implement the `make-statement-data` function in both `NormalPolicy` and `BuyTwoGetOneFreePolicy`.

We can easily mimic this design by using any one of the three approaches that we discussed in the section on the OCP. We could build vtables for the two abstractions. Or we could use `defprotocol` and `defrecord` to build actual Java interfaces and implementations. Or, finally, we could use multi-methods.

Let’s see what the multi-method approach looks like. Keep in mind that this is a child-sized problem posing as an adult situation. What you’ll see me do here is meant to show how much larger problems can be designed and partitioned.

In the end, as shown in [Figure 12.6](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch12.xhtml#ch12fig06), I split the whole system up into eleven modules, three of which are tests.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9780138176518/files/graphics/c12f006.jpg)

**Figure 12.6.** Splitting the Video Store application into modules

[Figure 12.6](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch12.xhtml#ch12fig06) looks like a UML diagram for an OO solution. The dependency inversion should be obvious. The `order-processing` module is the highest-level policy. It depends upon two abstractions. The `statement-formatter` is an interface, whereas the `statement-policy` is an abstract class with one implemented method.

If you are confused at my use of OO vernacular to describe a functional program in Clojure, you shouldn’t be. The OO words I’m using have very direct analogies in the functional world.

The `statement-formatter` interface is implemented by the `text-formatter` and the `HTML-formatter`. The `statement-policy` abstract class is implemented by the `normal-statement-policy`. The `buy-two-get-one-free-policy` implementation derives from `normal-statement-policy` but overrides one of its methods. The mechanisms behind all this “inheritance” will become clear in a moment.

The tests appear at the bottom. They are marked with `<T>`. They use a little utility module named `constructors` that knows how to build the basic data structures. Then each uses its particular portion of the production code to test what it needs.

Now let’s look at the source code. Pay special attention to the `ns` statements and notice that they match the arrows on the UML diagram.

Let’s begin with the `constructors`. They are pretty self-explanatory:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch12_images.xhtml#f169-01)

```
(ns video-store.constructors)

(defn make-customer [name]
  {:name name})

(defn make-movie [title type]
  {:title title
   :type type})

(defn make-rental [movie days]
  {:movie movie
   :days days})

(defn make-rental-order [customer rentals]
  {:customer customer
   :rentals rentals})
```

The `constructors` have no outgoing dependencies in the `ns` statement and simply build plain old Clojure data structures.

The integration test is in the `integration-specs` module:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch12_images.xhtml#f169-02)

```
(ns video-store.integration-specs
  (:require [speclj.core :refer :all]
            [video-store.constructors :refer :all]
            [video-store.text-statement-formatter :refer :all]
            [video-store.normal-statement-policy :refer :all]
            [video-store.order-processing :refer :all]))

(declare rental-order)

(describe "Integration Tests"
  (with rental-order (make-rental-order
                       (make-customer "Fred")
                       [(make-rental
                          (make-movie
                            "Plan 9 from Outer Space"
                            :regular)
                          1)
                        (make-rental
                          (make-movie "8 1/2", :regular)
                          2)
                        (make-rental
                          (make-movie "Eraserhead" :regular)
                          3)]))
  (it "formats a text statement"
    (should= (str "Rental Record for Fred\n"
                  "\tPlan 9 from Outer Space\t2.0\n"
                  "\t8 1/2\t2.0\n"
                  "\tEraserhead\t3.5\n"
                  "You owed 7.5\n"
                  "You earned 3 frequent renter points\n")
             (process-order
               (make-normal-policy)
               (make-text-formatter)
               @rental-order))))
```

This is pretty much the same as before, except that the `ns` statement has all the explicit source code dependencies. This test still violates the DIP, but only because it must call the `make-normal-policy` and `make-text-formatter` constructors within the corresponding modules. I suppose I could have used an _Abstract Factory_[26](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch12.xhtml#ch12fn26a) to break those last dependencies; but it didn’t seem worth the effort for a test that tests integration.

[26](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch12.xhtml#ch12fn26). See [Chapter 16](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch16.xhtml#ch16), “[Design Patterns Review](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch16.xhtml#ch16).”

The other two tests are more specific. Pay special attention to the fact that their source code dependencies only pull in what they need:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch12_images.xhtml#f170-01)

```
(ns video-store.statement-formatter-spec
  (:require [speclj.core :refer :all]
            [video-store.statement-formatter :refer :all]
            [video-store.text-statement-formatter :refer :all]
            [video-store.html-statement-formatter :refer :all]))

(declare statement-data)
(describe "Rental Statement Format"
  (with statement-data {:customer-name "CUSTOMER"
                        :movies [{:title "MOVIE"
                                  :price 9.9}]
                        :owed 100.0
                        :points 99})
  (it "Formats a text rental statement"
    (should= (str "Rental Record for CUSTOMER\n"
                  "\tMOVIE\t9.9\n"
                  "You owed 100.0\n"
                  "You earned 99 frequent renter points\n")
             (format-rental-statement
               (make-text-formatter)
               @statement-data
               )))

  (it "Formats an html rental statement"
      (should= (str
                 "<h1>Rental Record for CUSTOMER</h1>"
                 "<table>"
                 "<tr><td>MOVIE</td><td>9.9</td></tr>"
                 "</table>"
                 "You owed 100.0<br>"
                 "You earned <b>99</b> frequent renter points")
               (format-rental-statement
                 (make-html-formatter)
                 @statement-data))))
```

The `statement-formatter-spec` tests the two different formats. The format is specified by the first argument of the `format-rental-statement` function. That argument is created by the `make-text-formatter` and `make-html-formatter` functions, which are implemented in the appropriate modules, as you’ll see.

The last test is the `statement-policy-spec`:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch12_images.xhtml#f171-01)

```
(ns video-store.statement-policy-spec
  (:require
    [speclj.core :refer :all]
    [video-store.constructors :refer :all]
    [video-store.statement-policy :refer :all]
    [video-store.normal-statement-policy :refer :all]
    [video-store.buy-two-get-one-free-policy :refer :all]))

(declare customer normal-policy formatter)
(declare new-release-1 new-release-2 childrens)
(declare regular-1 regular-2 regular-3)

(describe "Rental Statement Calculation"
  (with customer (make-customer "CUSTOMER"))
  (with normal-policy (make-normal-policy))
  (with new-release-1 (make-movie "new release 1" :new-release))
  (with new-release-2 (make-movie "new release 2" :new-release))
  (with childrens (make-movie "childrens" :childrens))
  (with regular-1 (make-movie "regular 1" :regular))
  (with regular-2 (make-movie "regular 2" :regular))
  (with regular-3 (make-movie "regular 3" :regular))
  (context "normal policy"
    (it "makes statement for a single new release"
      (should= {:customer-name "CUSTOMER"
                :movies [{:title "new release 1"
                          :price 9.0}]
                :owed 9.0
                :points 2}
               (make-statement-data
                 @normal-policy
                 (make-rental-order
                   @customer
                   [(make-rental @new-release-1 3)]))))

    (it "makes statement for two new releases"
      (should= {:customer-name "CUSTOMER",
                :movies [{:title "new release 1", :price 9.0}
                         {:title "new release 2", :price 9.0}],
                :owed 18.0,
                :points 4}
               (make-statement-data
                 @normal-policy
                 (make-rental-order
                   @customer
                   [(make-rental @new-release-1 3)
                    (make-rental @new-release-2 3)]))))

    (it "makes statement for one childrens movie"
      (should= {:customer-name "CUSTOMER",
                :movies [{:title "childrens", :price 1.5}],
                :owed 1.5,
                :points 1}
               (make-statement-data
                 @normal-policy
                 (make-rental-order
                   @customer
                   [(make-rental @childrens 3)]))))

    (it "makes statement for several regular movies"
      (should= {:customer-name "CUSTOMER",
                :movies [{:title "regular 1", :price 2.0}
                         {:title "regular 2", :price 2.0}
                         {:title "regular 3", :price 3.5}],
                :owed 7.5,
                :points 3}
               (make-statement-data
                 @normal-policy
                 (make-rental-order
                   @customer
                   [(make-rental @regular-1 1)
                    (make-rental @regular-2 2)
                    (make-rental @regular-3 3)])))))

  (context "Buy two get one free policy"
    (it "makes statement for several regular movies"
      (should= {:customer-name "CUSTOMER",
                :movies [{:title "regular 1", :price 2.0}
                         {:title "regular 2", :price 2.0}
                         {:title "new release 1", :price 3.0}],
                :owed 5.0,
                :points 3}
               (make-statement-data
                 (make-buy-two-get-one-free-policy)
                 (make-rental-order
                   @customer
                   [(make-rental @regular-1 1)
                    (make-rental @regular-2 1)
                    (make-rental @new-release-1 1)]))))))
```

The `statement-policy-spec` tests the various pricing rules. You’ve seen the first batch already. The last test checks the buy two, get one free policy used by some stores. Notice that the policy is passed into the `make-statement-data` function and is created by the `make-normal-policy` and `make-buy-two-get-one-free-policy` functions.

Now, on to the production code. We begin with the `order-processing` module:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch12_images.xhtml#f174-01)

```
(ns video-store.order-processing
  (:require [video-store.statement-formatter :refer :all]
            [video-store.statement-policy :refer :all]))

(defn process-order [policy formatter order]
  (->> order
       (make-statement-data policy)
       (format-rental-statement formatter)))
```

There’s not much to it. Notice the source code dependencies only refer to the `statement-formatter` interface and the `statement-policy` abstraction.

The `statement-formatter` interface is very simple:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch12_images.xhtml#f174-02)

```
(ns video-store.statement-formatter)

(defmulti format-rental-statement
            (fn [formatter statement-data]
              (:type formatter)))
```

The `defmulti` statement is roughly equivalent to creating an abstract method in Java or C#. Since this module has nothing but one abstract method, it is roughly equivalent to an interface. The dispatcher function is trivial; it just returns the `:type` of the formatter.

The `statement-policy` abstraction is a bit more interesting:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch12_images.xhtml#f175-01)

```
(ns video-store.statement-policy)

(defn- policy-movie-dispatch [policy rental]
  [(:type policy) (-> rental :movie :type)])

(defmulti determine-amount policy-movie-dispatch)
(defmulti determine-points policy-movie-dispatch)
(defmulti total-amount (fn [policy _rentals] (:type policy)))
(defmulti total-points (fn [policy _rentals] (:type policy)))

(defn make-statement-data [policy rental-order]
  (let [{:keys [name]} (:customer rental-order)
        {:keys [rentals]} rental-order]
    {:customer-name name
     :movies (for [rental rentals]
               {:title (:title (:movie rental))
                :price (determine-amount policy rental)})
     :owed (total-amount policy rentals)
     :points (total-points policy rentals)}))
```

The `statement-policy` module has four abstract methods and one implemented method. Notice how it uses the Template Method[27](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch12.xhtml#ch12fn27a) pattern. Notice also that the `determine-amount` and `determine-points` functions use a dispatch code that is a tuple. That’s pretty interesting. It means that we can dispatch those functions based upon two degrees of freedom instead of one. That’s something that’s hard to do in most OO languages. We’ll see it used shortly.

[27](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch12.xhtml#ch12fn27). See [Chapter 17](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch17.xhtml#ch17), “[Wa-Tor](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch17.xhtml#ch17).”

But first let’s look at the `text-statement-formatter` implementation:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch12_images.xhtml#f175-02)

```
(ns video-store.text-statement-formatter
  (:require [video-store.statement-formatter :refer :all]))
(defn make-text-formatter [] {:type ::text})

(defmethod format-rental-statement
           ::text
           [_formatter statement-data]
  (let [customer-name (:customer-name statement-data)
        movies (:movies statement-data)
        owed (:owed statement-data)
        points (:points statement-data)]
    (str
      (format "Rental Record for %s\n" customer-name)
      (apply str
             (for [movie movies]
               (format "\t%s\t%.1f\n"
                 (:title movie)
                 (:price movie))))
      (format "You owed %.1f\n" owed)
      (format "You earned %d frequent renter points\n" points))))
```

This shouldn’t be much of a surprise. I just moved the code over here without much change. Notice the `make-text-formatter` function at the top.

The `html-statement-formatter` shouldn’t be very surprising either:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch12_images.xhtml#f176-01)

```
(ns video-store.html-statement-formatter
  (:require [video-store.statement-formatter :refer :all]))

(defn make-html-formatter [] {:type ::html})

(defmethod format-rental-statement ::html
  [formatter statement-data]
  (let [customer-name (:customer-name statement-data)
        movies (:movies statement-data)
        owed (:owed statement-data)
        points (:points statement-data)]
    (str
      (format "<h1>Rental Record for %s</h1>" customer-name)
      "<table>"
      (apply str
             (for [movie movies]
               (format "<tr><td>%s</td><td>%.1f</td></tr>"
                       (:title movie) (:price movie))))
      "</table>"
      (format "You owed %.1f<br>" owed)
      (format "You earned <b>%d</b> frequent renter points"
              points))))
```

The more interesting modules are the two policy modules. Let’s begin with `normal-statement-policy`:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch12_images.xhtml#f177-01)

```
(ns video-store.normal-statement-policy
  (:require [video-store.statement-policy :refer :all]))

(defn make-normal-policy [] {:type ::normal})

(defmethod determine-amount [::normal :regular] [_policy rental]
  (let [days (:days rental)]
    (if (> days 2)
      (+ 2.0 (* (- days 2) 1.5))
      2.0)))

(defmethod determine-amount
           [::normal :childrens]
           [_policy rental]
  (let [days (:days rental)]
    (if (> days 3)
      (+ 1.5 (* (- days 3) 1.5))
      1.5)))

(defmethod determine-amount
           [::normal :new-release]
           [_policy rental]
  (* 3.0 (:days rental)))

(defmethod determine-points [::normal :regular] [_policy _rental]
  1)

(defmethod determine-points
           [::normal :new-release]
           [_policy rental]
  (if (> (:days rental) 1) 2 1))

(defmethod determine-points
           [::normal :childrens]
           [_policy _rental]
  1)

(defmethod total-amount ::normal [policy rentals]
  (reduce + (map #(determine-amount policy %) rentals)))

(defmethod total-points ::normal [policy rentals]
  (reduce + (map #(determine-points policy %) rentals)))
```

That’s different, isn’t it? Look carefully at those `defmethod` statements. We’ve dispatched on both the policy type and the movie type. This isolates the business rules really well.

You might be worried that the two degrees of freedom will create an N*M problem, leading to a proliferation of the “determine” functions. You’ll see how I handle that in a minute.

Notice the `make-normal-policy` constructor at the top that was used by our tests.

Now let’s look at the `buy-two-get-one-free-policy` module:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch12_images.xhtml#f178-02)

```
(ns video-store.buy-two-get-one-free-policy
  (:require [video-store.statement-policy :refer :all]
            [video-store.normal-statement-policy :as normal]))

(derive ::buy-two-get-one-free ::normal/normal)

(defn make-buy-two-get-one-free-policy []
  {:type ::buy-two-get-one-free})

(defmethod total-amount
           ::buy-two-get-one-free
           [policy rentals]
  (let [amounts (map #(determine-amount policy %) rentals)]
    (if (> (count amounts) 2)
      (reduce + (drop 1 (sort amounts)))
      (reduce + amounts))))
```

Surprise, surprise! Look at that `derive` statement. This is Clojure’s way of allowing you to create ISA[28](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch12.xhtml#ch12fn28a) hierarchies. This statement says that a `::buy-two-get-one-free`[29](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch12.xhtml#ch12fn29a) policy is a `:normal` policy. The multi-method dispatching mechanism uses hierarchies like this to resolve which `defmethod` to dispatch to.

[28](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch12.xhtml#ch12fn28). Take care to avoid LSP violations!

[29](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch12.xhtml#ch12fn29). Once again, don’t worry about the double colons. They are just a way to scope keywords into a namespace.

What this says to the compiler is that it should use the `:normal` implementations unless overridden by a specific `::buy-two-get-one-free` implementation.

Thus, our module only has to override the `total-amount` function in order to subtract the least expensive movie if three or more are rented.

#### Conclusion

OK, that’s it. We’ve chopped this system up into 11 modules. Each module is nicely encapsulated. We have inverted the most important source code dependencies so that high-level policies do not depend upon low-level details.

The overall structure looks a lot like an OO program, and yet it is entirely functional.

Nice.