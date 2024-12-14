---
id: 01JF1NKZXQ62BCDQFY2PJZRHVM
modified: 2024-12-13T23:08:05-05:00
---
## 15

## Concurrency

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9780138176518/files/graphics/19_unnum_concurrency.jpg)

Concurrency in functional programs is substantially less complicated than it is in programs that support mutable state. The reason, as I said back in [Chapter 1](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch01.xhtml#ch01), is that you can’t have concurrent update problems if you don’t do updates. I also said that this means you can’t have race conditions.

These “facts” remove much of the complication of dealing with multiple threads. Threads simply cannot interfere with one another if they are composed of pure functions.

Or can they?

While comforting, those “facts” are not precisely true. The purpose of this chapter is to show how multithreaded “functional” programs can still have race conditions.

To examine this, let’s set up some interacting finite state machines. One of my favorite examples is the making of a telephone call in the 1960s. The sequence of events looked roughly like [Figure 15.1](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch15.xhtml#ch15fig01).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9780138176518/files/graphics/c15f001.jpg)

**Figure 15.1.** A message sequence chart of a telephone call

This is a _message sequence chart_. Time is on the vertical axis, and all messages are angled because they all take time to send.

You may be unfamiliar with the telephony nomenclature I used here. Indeed, if you were born after the year 2000, you may be unfamiliar with telephones in general. So, for the sake of history and nostalgia, let me walk you through the process.

Bob wants to place a call to Alice. Bob lifts the telephone receiver off its hook[1](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch15.xhtml#ch00fn1a) and holds it to his ear. The telephone company (telco) sends a dial tone[2](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch15.xhtml#ch00fn2a) to the receiver. Upon hearing that tone, Bob dials[3](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch15.xhtml#ch00fn3a) Alice’s number. The telco then sends a ringing voltage[4](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch15.xhtml#ch00fn4a) to Alice’s phone and a ringback[5](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch15.xhtml#ch00fn5a) tone to Bob’s receiver. Alice hears the ringing of her phone and lifts the receiver off the hook. The telco connects Bob to Alice, and Alice says “Hello” to Bob.

[1](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch15.xhtml#ch00fn1). Telephones in the early 20th century had a hook that the receiver hung on. By the 1960s, the hook had been replaced by a cradle that the receiver sat in; but it was still called the hook.

[2](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch15.xhtml#ch00fn2). This was a very recognizable sound that meant that the telephone system was ready for you to dial the number you wanted to call.

[3](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch15.xhtml#ch00fn3). The verb _dial_ means to enter the telephone number. In the early 1960s, this was accomplished by using a rotary dial on the face of the telephone.

[4](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch15.xhtml#ch00fn4). 90 volts in the United States.

[5](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch15.xhtml#ch00fn5). Another very distinct sound that was meant to entertain the caller while waiting for the called phone to be answered.

There are three finite state machines running in this scenario: Bob, telco, and Alice. Bob and Alice run separate instances of the User state machine[6](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch15.xhtml#ch00fn6a) shown in [Figure 15.2](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch15.xhtml#ch15fig02).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9780138176518/files/graphics/c15f002.jpg)

**Figure 15.2.** The User state machine

[6](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch15.xhtml#ch00fn6). These state machines are abbreviated to keep them simple. In reality, all the states would have transitions back to Idle.

The Telco state machine is shown in [Figure 15.3](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch15.xhtml#ch15fig03).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9780138176518/files/graphics/c15f003.jpg)

**Figure 15.3.** The Telco state machine

In these diagrams, the `->` symbol means to send the corresponding event to the other state machine.

So when Bob decides to make a call (the call event from the Idle state) the User state machine sends the off-hook event to the Telco. When the Telco is in the Waiting for Dial state and receives the Dial event from the User, it sends the Ring and Ringback events to the appropriate User state machines.

If you study these diagrams carefully, you should be able to see how the state machines and messages interact to allow Bob to call Alice.

We can write these state machines in Clojure quite simply:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch15_images.xhtml#f219pre01)

```
(def user-sm
  {:idle {:call [:calling caller-off-hook]
          :ring [:waiting-for-connection callee-off-hook]
          :disconnect [:idle nil]}
   :calling {:dialtone [:dialing dial]}
   :dialing {:ringback [:waiting-for-connection nil]}
   :waiting-for-connection {:connected [:talking talk]}
   :talking {:disconnect [:idle nil]}})

(def telco-sm
  {:idle {:caller-off-hook [:waiting-for-dial dialtone]
          :hangup [:idle nil]}
   :waiting-for-dial {:dial [:waiting-for-answer ring]}
   :waiting-for-answer {:callee-off-hook
                        [:waiting-for-hangup connect]}
   :waiting-for-hangup {:hangup [:idle disconnect]}})
```

Each state machine is simply a hash map of states, each of which contains a hash map of events that specify the new state and the action to be performed.

So when the `user-sm` is in the `:idle` state and it gets a `:call` event, it transitions to the `:calling` state and calls the `caller-off-hook` function.

These state machines can be executed by the following `transition` function:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch15_images.xhtml#f219pre02)

```
(defn transition [machine-agent event event-data]
  (swap! log conj (str (:name machine-agent) "<-" event))
  (let [state (:state machine-agent)
        sm (:machine machine-agent)
        result (get-in7 sm [state event])]
    (if (nil? result)
      (do
        (swap! log conj "TILT!")
        machine-agent)
      (do

        (when (second result)
          ((second result) machine-agent event-data))
        (assoc machine-agent :state (first result))))))
```

[7](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch15.xhtml#ch00fn7). The `get-in` function returns an element from a nested map. `(get-in {:a {:b 2}} [:a :b])` returns `2`.

The `log` variable is an `atom` that is simply used to accumulate a set of logging statements so that we can watch the operation of the state machines. Notice that this function takes the `machine-agent` and returns it with the new state in place. This means we can use it with Clojure’s `agent` STM facility.

An `agent` is initialized with a data structure and then serializes all updates to that data structure, thereby eliminating all concurrent update issues. Here are the functions that create the two different `agent`s:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch15_images.xhtml#f220pre01)

```
(defn make-user-agent [name]
  (agent {:state :idle :name name :machine user-sm}))

(defn make-telco-agent [name]
  (agent {:state :idle :name name :machine telco-sm}))
```

We send events to our agents by using the `agent`’s `send` function:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch15_images.xhtml#f220pre02)

```
(send caller transition :call [telco caller callee])
```

In this example, we are `send`ing the `transition` function to the `caller` agent. The `send` function returns immediately and queues up the `transition` function to be executed in the `agent`’s thread. The arguments to the `transition` function are the event (`:call`) and the data that should be passed to the action function. In this case, the data is a list of the three `agent`s that represent the finite state machines in the system.

The action functions are as follows:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch15_images.xhtml#f221pre01)

```
(defn caller-off-hook
  [sm-agent [telco caller callee :as call-data]]
  (swap! log conj (str  (:name @caller) " goes off hook."))
  (send telco transition :caller-off-hook call-data))

(defn dial [sm-agent [telco caller callee :as call-data]]
  (swap! log conj (str (:name @caller) " dials"))
  (send telco transition :dial call-data))

(defn callee-off-hook
  [sm-agent [telco caller callee :as call-data]]
  (swap! log conj (str (:name @callee) " goes off hook"))
  (send telco transition :callee-off-hook call-data))

(defn talk [sm-agent [telco caller callee :as call-data]]
  (swap! log conj (str (:name sm-agent) " talks."))
  (Thread/sleep 10)
  (swap! log conj (str (:name sm-agent) " hangs up."))
  (send telco transition :hangup call-data))

(defn dialtone [sm-agent [telco caller callee :as call-data]]
  (swap! log conj (str "dialtone to " (:name @caller)))
  (send caller transition :dialtone call-data))

(defn ring [sm-agent [telco caller callee :as call-data]]
  (swap! log conj (str "telco rings " (:name @callee)))
  (send callee transition :ring call-data)
  (send caller transition :ringback call-data))

(defn connect [sm-agent [telco caller callee :as call-data]]
  (swap! log conj "telco connects")
  (send caller transition :connected call-data)
  (send callee transition :connected call-data))

(defn disconnect [sm-agent [telco caller callee :as call-data]]
  (swap! log conj "disconnect")
  (send callee transition :disconnect call-data)
  (send caller transition :disconnect call-data))
```

The second argument in each of the action functions is _destructured_.[8](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch15.xhtml#ch00fn8a) So, for example, the `call-data` sent to `caller-off-hook` is a list, the first element of which will be placed in `telco`, the second in `caller`, the third in `callee`, and the whole list in `call-data`.

[8](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch15.xhtml#ch00fn8). In short, destructuring is a convenient way of breaking a complex data element into named components. See the Clojure documentation for more details.

Given this implementation, we should be able to make a call between Bob and Alice by executing the following code. I have written it in the form of a test:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch15_images.xhtml#f222pre01)

```
(it "should make and receive call"
  (let [caller (make-user "Bob")
        callee (make-user "Alice")
        telco (make-telco "telco")]
    (reset! log [])
    (send caller transition :call [telco caller callee])
    (Thread/sleep 100)
    (prn @log)
    (should= :idle (:state @caller))
    (should= :idle (:state @callee))
    (should= :idle (:state @telco))))
```

This test passes, which means that all the state machines returned to the idle state by the time 100ms had passed. The log output looks like this:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch15_images.xhtml#f222pre02)

```
"Bob<-:call" "Bob goes off hook"
"telco<-:caller-off-hook" "dialtone to Bob"
"Bob<-:dialtone" "Bob dials"
"telco<-:dial" "telco rings Alice"
"Alice<-:ring" "Alice goes off hook"
"Bob<-:ringback"
"telco<-:callee-off-hook" "telco connects"
"Bob<-:connected" "Bob talks"
"Alice<-:connected" "Alice talks"
"Bob hangs up"
"Alice hangs up"
"telco<-:hangup" "disconnect"
"Alice<-:disconnect"
"Bob<-:disconnect"
"telco<-:hangup"
```

You can see how the threads interleaved with one another, while all three finite state machines worked together to drive the call to a successful completion.

The three agents have mutable state; but there can be no concurrent update problems because the agents serialize their operations. So no race conditions, right?

Not so fast there, Newt. Let’s investigate another scenario.

What I’m about to show you in [Figure 15.4](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch15.xhtml#ch15fig04), is a race condition that existed in the telephone system in the ‘60s.[9](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch15.xhtml#ch00fn9a) Once again, we begin with Bob calling Alice. But this time Alice is just about to call Bob.

[9](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch15.xhtml#ch00fn9). It probably still exists today if you use landlines.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9780138176518/files/graphics/c15f004.jpg)

**Figure 15.4.** The race condition in the telephone system

Do you see what went wrong? Those crossed lines are the problem. That’s a race condition. The telco tried to ring Alice’s phone; but before it could make the sound, Alice picked up the receiver in order to call Bob. From the point of view of the telco, everything is fine. It rang the phone and Alice picked up. So the telco happily connects Bob and Alice. But Alice is sitting there waiting for a dial tone; and Bob is confused because nobody has said hello and the ringback tone has stopped.

The most likely outcome is that both parties hang up without talking to each other. Alternatively, Alice might say something and Bob might respond, and they’d get into the comic routine of who called who.

Can we make our state machines emulate this fault? Here’s the setup, once again posed as a test:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch15_images.xhtml#f224pre01)

```
(it "should race"
  (let [caller (make-user "Bob")
        callee (make-user "Alice")
        telco1 (make-telco "telco1")
        telco2 (make-telco "telco2")]
    (reset! log [])
    (send caller transition :call [telco1 caller callee])
    (send callee transition :call [telco2 callee caller])
    (Thread/sleep 100)
    (prn @log)
    (should= :idle (:state @caller))
    (should= :idle (:state @callee))
    (should= :idle (:state @telco1))
    (should= :idle (:state @telco2))))
```

Notice that we now have four state machines: one for Bob, one for Alice, and one telco for each of the two calls. The test fails. After 100ms, the state machines have not returned to the Idle state.

So, what does the log tell us?

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch15_images.xhtml#f224pre02)

```
"Bob<-:call" "Bob goes off hook"
"telco1<-:caller-off-hook"
"Alice<-:call" "Alice goes off hook"
"telco2<-:caller-off-hook"
"dialtone to Bob"
"Bob<-:dialtone" "Bob dials"
"telco1<-:dial" "telco rings Alice"
"Bob<-:ringback"
"Alice<-:ring" "TILT!" …
```

This took me several tries, because the window for that particular race condition is pretty narrow. But there it is. See that `TILT!`? That’s what our `transition` function puts in the log if it is ever asked to make an invalid transition. Alice is still in the `:calling` state waiting for the `:dialtone` event, and has no way to deal with the `:ring` event.

The bottom line is that race conditions are still possible even though concurrent updates are not. That’s because it is always possible to construct interacting state machines that get out of sync with one another.

### Conclusion

Somewhere around the turn of the century, Moore’s law died. Clock rates hit a maximum of about 3GHz and then just stopped increasing. To drive more throughput, hardware engineers started putting more processors on their chips. We went through the dual-core stage and the quad-core stage—and we thought we were going to see a doubling in cores every other year or so. We started to fret about the possibility of dealing with machines that had 32, or 64, or 128 cores.

This is about the time functional languages started to gain in popularity. The thought was that since functional programs don’t mutate data, multicore operations would be made much simpler. If you are working with pure functions, it is theoretically easy to spread those functions out over a plethora of cores.

But Moore’s law wasn’t done dying. It died for clock speed a few years before it died for component density. So, for the past decade or more, our processors have been quad core (don’t talk to me about hyperthreading); and that is not likely to change. This has decreased the fear of the 128-core processor and lessened the urgency behind functional programming.

And that’s probably a good thing because, as this chapter has shown, the reasoning was somewhat faulty to begin with. Race conditions might be more common in threads that have mutable variables, but in any system where there are concurrent finite state machines, the possibility exists that race conditions might drive them out of sync with one another.