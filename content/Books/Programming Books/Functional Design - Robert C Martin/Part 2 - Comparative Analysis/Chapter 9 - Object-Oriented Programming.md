---
id: 01JF1ND6MMSE0YWD4N56H5XKQK
modified: 2024-12-13T23:04:23-05:00
---
## 9

## Object-Oriented Programming

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9780138176518/files/graphics/09_unnum_objectorientedprogramming.jpg)

In the preceding chapter, we saw that the OO style of programming is strongly related to data types and the cohesion of data. But that’s not all there is to object orientation. Indeed, data cohesion may be secondary to another attribute of object orientation: polymorphism.

In _Clean Architecture_,[1](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch09.xhtml#ch09fn1a) I made the point that the OO style has three attributes: encapsulation, inheritance, and polymorphism. I then led you through the reasoning that, of the three, polymorphism is the most beneficial. The other two are, at best, ancillary.

[1](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch09.xhtml#ch09fn1). Robert C. Martin, _Clean Architecture_ (Pearson, 2017).

The examples in the previous chapters did not lend themselves to any polymorphism. Let’s correct that by examining how we might solve the Payroll problem from Section 3 of _Agile Software Development: Principles, Patterns, and Practices_.[2](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch09.xhtml#ch09fn2a)

[2](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch09.xhtml#ch09fn2). Robert C Martin, _Agile Software Development: Principles, Patterns, and Practices_ (Pearson, 2002).

The requirements are as follows.

- There is a database of employee records.
    
- The payroll program runs daily, generating payments for those employees who should be paid on that day.
    
- Salaried employees are paid on the last business day of the month. Their monthly salary is a field in their employee record.
    
- Commissioned employees are paid every other Friday. They are paid a base salary plus commission. The base salary and the commission rate are fields in their employee record. Commission is calculated by multiplying the commission rate by the total of the sales receipts for that employee.
    
- Hourly employees are paid every Friday. Their hourly rate is a field in their employee record. Their pay is calculated by multiplying their hourly rate by the sum of the hours on their timecards for the week. If that sum is greater than 40, the remaining hours are paid at 1.5 times their hourly rate.
    
- Employees are given the option to have their paychecks mailed to their home address, held at their paymaster’s office, or directly deposited into their bank account. The address, paymaster, and bank information are fields in their employee record.
    

The typical OO solution to this problem is shown in the unified modeling language (UML) diagram in [Figure 9.1](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch09.xhtml#ch09fig01).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9780138176518/files/graphics/c09f001.jpg)

**Figure 9.1.** Object model for the Payroll problem

Perhaps the best place to begin is with the `Payroll` class. In Java, it has a `run` method that looks like this:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch09_images.xhtml#f097-01)

```
void run() {
  for (Employee e : db.getEmployees()) {
    if (e.isPayDay()) {
      Pay pay = e.calcPay();
      e.sendPay(pay);
    }
  }
}
```

I have made the point many times, and in many places, including the aforementioned books, that this little snippet of code is the _pure truth_. For each employee, if today is the day they should be paid, then calculate their pay and send it to them.

From that little snippet of code, the rest of the implementation ought to be pretty clear. There are three uses of the _Strategy_[3](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch09.xhtml#ch09fn3a) pattern: one to implement `calcPay`, another to implement `isPayDay`, and the last to implement `sendPay`.

[3](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch09.xhtml#ch09fn3). Erich Gamma, Richard Helm, Ralph Johnson, and John Vlissides, _Design Patterns: Elements of Reusable Object-Oriented Software_ (Addison-Wesley, 1995), 315.

It should also be clear that this structure of objects must be built up by the `getEmployees` function, which reads the employees from the database and arranges them properly. It is unlikely that the data in the database looks like the object structure seen here.

There is also a very clear architectural boundary (dashed line) that cuts across all those inheritance relationships, dividing the high-level abstractions from the low-level details.

### Functional Payroll

[Figure 9.2](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch09.xhtml#ch09fig02) shows what this might look like as a functional program.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9780138176518/files/graphics/c09f002.jpg)

**Figure 9.2.** Data flow diagram of the Payroll problem

Isn’t it interesting that I chose a data flow diagram (DFD) to represent the functional solution? DFDs are very helpful in depicting the relationships between processes and data elements, but they are not nearly as helpful as UML class diagrams when it comes to depicting architectural decisions.

Still, the DFD helps us propose the functional version of the _pure truth_:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch09_images.xhtml#f099-01)

```
(defn payroll [today db]
  (let [employees (get-employees db)
        employees-to-pay (get-employees-to-be-paid-today
                            today employees)
        amounts (get-paycheck-amounts employees-to-pay)
        ids (get-ids employees-to-pay)
        dispositions (get-dispositions employees-to-pay)]
    (send-paychecks ids amounts dispositions)))
```

Notice that this differs from the Java version in that it is not an iterative approach. Rather, the list of employees flows through the program, getting modified at each stage according to the data flow diagram. This is typical of the way functional programs are conceived and written. Functional programs tend to be more like _plumbing_ than step-by-step procedures. They regulate and modify the flow of data, rather than iterating step by step through the data.

So, what about the architecture? There was that nice architectural boundary in the UML diagram of the OO version. Where is the architectural boundary in the functional version?

Let’s look a bit deeper. The tests may give us some hints:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch09_images.xhtml#f100-01)

```
(it "pays one salaried employee at end of month by mail"
  (let [employees [{:id "emp1"
                    :schedule :monthly
                    :pay-class [:salaried 5000]
                    :disposition [:mail "name" "home"]}]
        db {:employees employees}
        today (parse-date "Nov 30 2021")]
    (should= [{:type :mail
               :id "emp1"
               :name "name"
               :address "home"
               :amount 5000}]
             (payroll today db))))
```

In this test, the database contains a list of `employee`s, and each `employee` is a hash map with specific fields. That’s not so different from an object, is it? The `payroll` function returns a list of paycheck directives, each of which is also a hash map—another object. Interesting.

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch09_images.xhtml#f100-02)

```
(it "pays one hourly employee on Friday by Direct Deposit"
  (let [employees [{:id "empid"
                    :schedule :weekly
                    :pay-class [:hourly 15]
                    :disposition [:deposit "routing" "account"]}]
        time-cards {"empid" [["Nov 12 2022" 80/104]]}
        db {:employees employees :time-cards time-cards}
        friday (parse-date "Nov 18 2022")]
    (should= [{:type :deposit
               :id "empid"
               :routing "routing"
               :account "account"
               :amount 120}]
             (payroll friday db))))
```

[4](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch09.xhtml#ch09fn4). This is not 80 divided by 10. Rather, it is the rational number 80/10. This ensures that subsequent mathematics will not treat the value as an integer.

This test shows how the `employee` and `paycheck-directive` objects vary based upon the `:schedule`, `:pay-class`, and `:disposition`. It also shows that the database contains `time-card`s associated with employee `id`s. From this, the third test ought to be predictable:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch09_images.xhtml#f101-01)

```
(it "pays one commissioned employee on an even Friday by Paymaster"
  (let [employees [{:id "empid"
                    :schedule :biweekly
                    :pay-class [:commissioned 100 5/100]
                    :disposition [:paymaster "paymaster"]}]
        sales-receipts {"empid" [["Nov 12 2022" 15000]]}
        db {:employees employees :sales-receipts sales-receipts}
        friday (parse-date "Nov 18 2022")]
    (should= [{:type :paymaster
               :id "empid"
               :paymaster "paymaster"
               :amount 850}]
             (payroll friday db))))
```

Notice that the payments are being properly calculated, the dispositions are being correctly interpreted, and—as far as we can tell—the schedules are being followed. So how is this all being accomplished?

Here’s the key to it all:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch09_images.xhtml#f102-01)

```
(defn get-pay-class [employee]
  (first (:pay-class employee)))

(defn get-disposition [paycheck-directive]
  (first (:disposition paycheck-directive)))

(defmulti is-today-payday :schedule)
(defmulti calc-pay get-pay-class)
(defmulti dispose get-disposition)

(defn get-employees-to-be-paid-today [today employees]
  (filter5 #(is-today-payday % today) employees))

(defn- build-employee [db employee]
  (assoc employee :db db))

(defn get-employees [db]
  (map (partial6 build-employee db) (:employees db)))

(defn create-paycheck-directives [ids payments dispositions]
  (map #(assoc {} :id %1 :amount %2 :disposition %3)
       ids payments dispositions))

(defn send-paychecks [ids payments dispositions]
  (for7 [paycheck-directive
        (create-paycheck-directives ids payments dispositions)]
    (dispose paycheck-directive)))

(defn get-paycheck-amounts [employees]
  (map calc-pay employees))

(defn get-dispositions [employees]
  (map :disposition employees))

(defn get-ids [employees]
  (map :id employees))
```

[5](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch09.xhtml#ch09fn5). `(filter predicate list)` calls `predicate` for every member of `list` and returns a sequence of all the members for which `predicate` was not _falsey_.

[6](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch09.xhtml#ch09fn6). The `partial` function takes a function and some arguments, and returns a new function in which all those arguments have already been initialized. Thus, `((partial f 1) 2)` is equivalent to `(f 1 2)`.

[7](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch09.xhtml#ch09fn7). In this case, the `for` function calls `dispose` for each `paycheck-directive` in the list returned by `create-paycheck-directives`.

Do you see those `defmulti` statements (in bold)? They are analogous, though not identical, to a Java interface. Each `defmulti` defines a polymorphic function. However, that function does not dispatch based upon an intrinsic type, the way Java or C# or even Ruby and Python do. Rather, they dispatch upon the result of the function specified right after the name.

So, the `get-pay-class` function returns the value that the `calc-pay` function will polymorphically dispatch on. What does `get-pay-class` return? It returns the first element of the `pay-class` field of the `employee`. According to our tests, those values are `:salaried`, `:hourly`, and `:commissioned`.

So where are the implementations of the `calc-pay` functions? They are _further down_ in the program:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch09_images.xhtml#f103-01)

```
(defn-8 get-salary [employee]
  (second (:pay-class employee)))

(defmethod calc-pay :salaried [employee]
  (get-salary employee))

(defmethod calc-pay :hourly [employee]
  (let [db (:db employee)
        time-cards (:time-cards db)
        my-time-cards (get9 time-cards (:id employee))
        [_ hourly-rate]10 (:pay-class employee)
        hours (map second my-time-cards)
        total-hours (reduce + hours)]
    (* total-hours hourly-rate)))

(defmethod calc-pay :commissioned [employee]
  (let [db (:db employee)
        sales-receipts (:sales-receipts db)
        my-sales-receipts (get sales-receipts (:id employee))
        [_ base-pay commission-rate] (:pay-class employee)
        sales (map second my-sales-receipts)
        total-sales (reduce + sales)]
    (+ (* total-sales commission-rate) base-pay)))
```

[8](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch09.xhtml#ch09fn8). The trailing `-` makes this a private function, so only functions in this file can access it.

[9](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch09.xhtml#ch09fn9). `(get m k)` returns the value of `k` in the map `m`.

[10](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch09.xhtml#ch09fn10). _Destructures_ the `pay-class` of the `employee` and ignores the first element.

I italicized the words _further down_ because that is significant in a Clojure program. Clojure programs cannot call functions that are declared below the point of call. But these functions _are_ declared below the point of call. That means there is a source code dependency inversion. The `calc-pay` implementations are called by the `payroll` function; but the `payroll` function is above the `calc-pay` implementations.

Indeed, I could move all the implementations of the `defmulti` function to a different source file that the `payroll` source file does not `require`.

If we draw the relationships between those source files, we get the diagram in [Figure 9.3](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch09.xhtml#ch09fig03).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9780138176518/files/graphics/c09f003.jpg)

**Figure 9.3.** Dependency inversion

The arrows depict the `requires` relationships between the source files. The source code of those `requires` in the `payroll-implementation.clj` file looks like this:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch09_images.xhtml#f105-01)

```
(ns payroll-implementation
  (:require [payroll :refer [is-today-payday calc-pay dispose]]))
```

The source code dependency inversion should be obvious. The `payroll` function in `payroll.clj` calls the `is-today-payday`, `calc-pay`, and `dispose` implementations in the `payroll-implementation.clj` file, but the `payroll.clj` file does not depend upon the `payroll-implementation.clj` file. The dependency points the other way around.

What does all this inversion mean? It means that the low-level details in `payroll-implementation.clj` depend upon the high-level policy in `payroll.clj`. And whenever low-level details depend upon high-level policy, we have the potential for an architectural boundary. We could even draw it as shown in [Figure 9.4](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch09.xhtml#ch09fig04).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9780138176518/files/graphics/c09f004.jpg)

**Figure 9.4.** Architectural boundary

Notice that I used a UML implements arrow. It’s almost as if `Payroll` and `PayrollImplementation` were classes in a Java program.

But we can do even better than this. We can move all the `defmulti` statements, along with their supporting functions, into their own `payroll-interface` namespace and source file, like this:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch09_images.xhtml#f106-01)

```
(ns payroll-interface)

(defn- get-pay-class [employee]
  (first (:pay-class employee)))

(defn- get-disposition [paycheck-directive]
  (first (:disposition paycheck-directive)))

(defmulti is-today-payday :schedule)
(defmulti calc-pay get-pay-class)
(defmulti dispose get-disposition)
```

And now we can draw the architecture diagram as shown in [Figure 9.5](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch09.xhtml#ch09fig05).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9780138176518/files/graphics/c09f005.jpg)

**Figure 9.5.** Architecture with interface

This is starting to look more and more like the UML diagram of a Java or C# program. It looks like we got a `Payroll` class, a `PayrollInterface` class, and a `PayrollImplementation` class. And indeed, from an architectural point of view, that’s a pretty accurate statement.

But there are some interesting differences. Where, for example, are the `PaySchedule`, `PayClassification`, and `PayDisposition` classes that we saw in the UML of the OO Java program?

We could easily pull them out of the Clojure program by splitting the `PayrollImplementation.clj` file into three namespaces and files, as shown in [Figure 9.6](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch09.xhtml#ch09fig06).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9780138176518/files/graphics/c09f006.jpg)

**Figure 9.6.** Split architecture

This is not the kind of thing you can do in Java or C# since there is no way, in those languages, to implement each function of an interface in a different module. However, it’s perfectly possible in Clojure. The important thing to remember is that this is an _architectural_ diagram, not a class diagram. `PaySchedule`, `PayClassification`, and `PayDisposition` are namespaces and source files, not classes. We do not make instances of them. They don’t represent objects in an OO sense.

Not that there aren’t objects in our Clojure solution. There certainly are. The `employee`, the `paycheck-directive`, and even the `pay-class` and `disposition` are objects. They do not have methods as strongly associated with them as they would if they were written in an OO language; but there are functions through which those objects flow.

### Namespaces and Source Files

In Clojure especially, namespaces and source files are deeply connected. Each namespace must be contained in its own source file, and the name of that file must correspond[11](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch09.xhtml#ch09fn11a) to the name of the namespace. This is very similar to the way Java forces public classes into their own source file named for the class. It is also very similar to the file/class convention used by C++ and C# programmers. This could lead you to consider that each Clojure namespace is something like a class.

[11](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch09.xhtml#ch09fn11). Through a simple translation algorithm.

The correspondence is not perfect, of course. The contents of a Clojure namespace need not be class-like at all. But, in general, the concept is not a bad one.

One of the great temptations in functional languages like Clojure is to group functions into namespaces in a kind of ad hoc, by-feel way. Without the OO structure to force us to divide functions into classes that exist in their own source files, we often wind up with source file structures that are ricketier and more fragile than they ought to be.

So, when writing functional programs, it is not a bad idea to consider the partitioning disciplines of OO and continue to apply them. We’ll see more of this later as we investigate principles, patterns, and architecture.

### Conclusion

First of all, functional programs and OO programs are different. Functional programs tend to be constructions of plumbing that regulate data flow transformations, while mutable OO programs tend to iterate step by step over objects. However, from an architectural point of view, the two styles are quite compatible. It turns out that we can partition the functions of a functional program into the same kinds of architecturally significant elements as an OO program. From an architectural point of view, there’s very little difference.

Functional programs may not be composed of syntactically enforced classes that enclose methods and define objects. Yet, objects still exist in functional programs. Those objects are less tightly bound to the functions that operate upon them than they would be in an OO language. Whether that is an advantage or a disadvantage is something we will continue to probe in the chapters that follow.

We shall see as these pages turn more and more toward design and architecture, the differences between functional programs and the object orientation of immutable objects start to become less and less relevant.