---
id: 01JF1NCG59A6VNTB6DKK0HE0PK
modified: 2024-12-13T23:04:00-05:00
---
## 8

## Gossiping Bus Drivers

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9780138176518/files/graphics/08_unnum_gossipingbusdrivers.jpg)

So far in this comparative analysis we haven’t found a strong reason to prefer functional programming over OO programming. So let’s examine a slightly more interesting problem.

Object orientation was born in 1966 when Ole-Johan Dahl and Kristen Nygaard added some modifications to the ALGOL-60 language in order to make the language more amenable to discrete event simulation.[1](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch08.xhtml#ch08fn1a) The new language was called SIMULA-67 and is considered to be the first true OO programming language.

[1](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch08.xhtml#ch08fn1). Legend has it that they were simulating Norwegian ocean shipping.

So let’s do a comparative analysis of a simple discrete event simulator. That should keep the problem squarely in the OO wheelhouse. A nice problem to choose is the Gossiping Bus Drivers kata.[2](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch08.xhtml#ch08fn2a)

[2](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch08.xhtml#ch08fn2). [https://kata-log.rocks/gossiping-bus-drivers-kata](https://kata-log.rocks/gossiping-bus-drivers-kata)

Given _n_ drivers, each with their own circular route of stops, determine how many steps are required until all gossip known to each bus driver is known by all. Drivers only gossip if they arrive together at the same stop.

So, let’s say that Bob knows rumor X and drives route [p,q,r]. Jim knows rumor Y and drives route [s,t,u,p]. When will Bob and Jim be able to share their gossip? If they start at time 0, then at time 3 they will both be at stop p; remember, the routes are circular.

The process is limited to 480 steps.

This problem gets more interesting when there are more than two drivers and more complex routes.

### Java Solution

I wrote a solution to this problem in Java. I started out with a very traditional kind of OO analysis and design (see [Figure 8.1](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch08.xhtml#ch08fig01)).

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9780138176518/files/graphics/c08f001.jpg)

**Figure 8.1.** Simple object model for the Java version

The Simulator holds many Drivers. Each Driver has a Route, and each Route contains many Stops. Each Stop has many Drivers, and each Driver has many Rumors.

This is a fairly simple object model. There’s not even any inheritance or polymorphism implied. So it should be a pretty straightforward implementation.

I wrote the Java code using TDD, of course. Here are the tests. As you can see, they are fairly wordy; but at least they all fit into a single test class:[3](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch08.xhtml#ch08fn3a).

[3](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch08.xhtml#ch08fn3). If you read my book _Clean Craftsmanship_ (Addison-Wesley, 2021), you’ll understand why this is a good thing.

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch08_images.xhtml#f079-01)

```
package gossipingBusDrivers;

import org.junit.Before;
import org.junit.Test;

import static org.hamcrest.MatcherAssert.assertThat;
import static org.hamcrest.collection.IsEmptyCollection.empty;

import static org.hamcrest.collection.
  IsIterableContainingInAnyOrder.containsInAnyOrder;
import static org.junit.Assert.assertEquals;

public class GossipTest {
  private Stop stop1;
  private Stop stop2;
  private Stop stop3;
  private Route route1;
  private Route route2;
  private Rumor rumor1;
  private Rumor rumor2;
  private Rumor rumor3;
  private Driver driver1;
  private Driver driver2;

  @Before
  public void setUp() {
    stop1 = new Stop("stop1");
    stop2 = new Stop("stop2");
    stop3 = new Stop("stop3");
    route1 = new Route(stop1, stop2);
    route2 = new Route(stop1, stop2, stop3);
    rumor1 = new Rumor("Rumor1");
    rumor2 = new Rumor("Rumor2");
    rumor3 = new Rumor("Rumor3");
    driver1 = new Driver("Driver1", route1, rumor1);
    driver2 = new Driver("Driver2", route2, rumor2, rumor3);
  }

  @Test
  public void driverStartsAtFirstStopInRoute() throws Exception {
    assertEquals(stop1, driver1.getStop());
  }

  @Test
  public void driverDrivesToNextStop() throws Exception {
    driver1.drive();
    assertEquals(stop2, driver1.getStop());
  }

  @Test
  public void driverReturnsToStartAfterLastStop()
  throws Exception {
    driver1.drive();
    driver1.drive();
    assertEquals(stop1, driver1.getStop());
  }

  @Test
  public void firstStopHasDriversAtStart() throws Exception {
    assertThat(stop1.getDrivers(), containsInAnyOrder(driver1,
                                                      driver2));
    assertThat(stop2.getDrivers(), empty());
  }

  @Test
  public void multipleDriversEnterAndLeaveStops()
  throws Exception {
    assertThat(stop1.getDrivers(), containsInAnyOrder(driver1,
                                                      driver2));
    assertThat(stop2.getDrivers(), empty());
    assertThat(stop3.getDrivers(), empty());
    driver1.drive();
    driver2.drive();
    assertThat(stop1.getDrivers(), empty());
    assertThat(stop2.getDrivers(), containsInAnyOrder(driver1,
                                                      driver2));
    assertThat(stop3.getDrivers(), empty());
    driver1.drive();
    driver2.drive();
    assertThat(stop1.getDrivers(), containsInAnyOrder(driver1));
    assertThat(stop2.getDrivers(), empty());
    assertThat(stop3.getDrivers(), containsInAnyOrder(driver2));
    driver1.drive();
    driver2.drive();
    assertThat(stop1.getDrivers(), containsInAnyOrder(driver2));
    assertThat(stop2.getDrivers(), containsInAnyOrder(driver1));
    assertThat(stop3.getDrivers(), empty());
  }

  @Test
  public void driversHaveRumorsAtStart() throws Exception {
    assertThat(driver1.getRumors(), containsInAnyOrder(rumor1));
    assertThat(driver2.getRumors(), containsInAnyOrder(rumor2,
                                                       rumor3));
  }

  @Test
  public void noDriversGossipAtEmptyStop() throws Exception {
    stop2.gossip();
    assertThat(driver1.getRumors(), containsInAnyOrder(rumor1));
    assertThat(driver2.getRumors(), containsInAnyOrder(rumor2,
                                                       rumor3));
  }

  @Test
  public void driversGossipAtStop() throws Exception {
    stop1.gossip();
    assertThat(driver1.getRumors(), containsInAnyOrder(rumor1,
                                                       rumor2,
                                                       rumor3));

    assertThat(driver2.getRumors(), containsInAnyOrder(rumor1,
                                                       rumor2,
                                                       rumor3));
  }

  @Test
  public void gossipIsNotDuplicated() throws Exception {
    stop1.gossip();
    stop1.gossip();
    assertThat(driver1.getRumors(), containsInAnyOrder(rumor1,
                                                       rumor2,
                                                       rumor3));

    assertThat(driver2.getRumors(), containsInAnyOrder(rumor1,
                                                       rumor2,
                                                       rumor3));
  }

  @Test
  public void driveTillEqualTest() throws Exception {
    assertEquals(1, Simulation.driveTillEqual(driver1,
                                              driver2));
  }

  @Test
  public void acceptanceTest1() throws Exception {
    Stop s1 = new Stop("s1");
    Stop s2 = new Stop("s2");
    Stop s3 = new Stop("s3");
    Stop s4 = new Stop("s4");
    Stop s5 = new Stop("s5");
    Route r1 = new Route(s3, s1, s2, s3);
    Route r2 = new Route(s3, s2, s3, s1);
    Route r3 = new Route(s4, s2, s3, s4, s5);
    Driver d1 = new Driver("d1", r1, new Rumor("1"));
    Driver d2 = new Driver("d2", r2, new Rumor("2"));
    Driver d3 = new Driver("d3", r3, new Rumor("3"));
    assertEquals(6, Simulation.driveTillEqual(d1, d2, d3));
  }

  @Test
  public void acceptanceTest2() throws Exception {
    Stop s1 = new Stop("s1");
    Stop s2 = new Stop("s2");
    Stop s5 = new Stop("s5");
    Stop s8 = new Stop("s8");
    Route r1 = new Route(s2, s1, s2);
    Route r2 = new Route(s5, s2, s8);
    Driver d1 = new Driver("d1", r1, new Rumor("1"));
    Driver d2 = new Driver("d2", r2, new Rumor("2"));
    assertEquals(480, Simulation.driveTillEqual(d1, d2));
  }
}
```

The solution code is broken up into several small files.

#### Driver

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch08_images.xhtml#f084-01)

```
package gossipingBusDrivers;

import java.util.Arrays;
import java.util.HashSet;
import java.util.Set;

public class Driver {
  private String name;
  private Route route;
  private int stopNumber = 0;
  private Set<Rumor> rumors;

  public Driver(String name, Route theRoute,
                Rumor... theRumors) {
    this.name = name;
    route = theRoute;
    rumors = new HashSet<>(Arrays.asList(theRumors));
    route.stopAt(this, stopNumber);
  }

  public Stop getStop() {
    return route.get(stopNumber);
  }

  public void drive() {
    route.leave(this, stopNumber);
    stopNumber = route.getNextStop(stopNumber);
    route.stopAt(this, stopNumber);
  }

  public Set<Rumor> getRumors() {
    return rumors;
  }

  public void addRumors(Set<Rumor> newRumors) {
    rumors.addAll(newRumors);
  }
}
```

#### Route

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch08_images.xhtml#f085-01)

```
package gossipingBusDrivers;

public class Route {
  private Stop[] stops;

  public Route(Stop... stops) {
    this.stops = stops;
  }

  public Stop get(int stopNumber) {
    return stops[stopNumber];
  }

  public int getNextStop(int stopNumber) {
    return (stopNumber + 1) % stops.length;
  }

  public void stopAt(Driver driver, int stopNumber) {
    stops[stopNumber].addDriver(driver);
  }

  public void leave(Driver driver, int stopNumber) {
    stops[stopNumber].removeDriver(driver);
  }
}
```

#### Stop

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch08_images.xhtml#f085-02)

```
package gossipingBusDrivers;

import java.util.ArrayList;
import java.util.HashSet;
import java.util.List;
import java.util.Set;

public class Stop {
  private String name;
  private List<Driver> drivers = new ArrayList<>();

  public Stop(String name) {
    this.name = name;
  }

  public String toString() {
    return name;
  }

  public List<Driver> getDrivers() {
    return drivers;
  }

  public void addDriver(Driver driver) {
    drivers.add(driver);
  }

  public void removeDriver(Driver driver) {
    drivers.remove(driver);
  }

  public void gossip() {
    Set<Rumor> rumorsAtStop = new HashSet<>();
    for (Driver d : drivers)
      rumorsAtStop.addAll(d.getRumors());
    for (Driver d : drivers)
      d.addRumors(rumorsAtStop);
  }
}
```

#### Rumor

```
package gossipingBusDrivers;

public class Rumor {
  private String name;
  public Rumor(String name) {
    this.name = name;
  }

  public String toString() {
    return name;
  }
}
```

#### Simulation

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch08_images.xhtml#f087-01)

```
package gossipingBusDrivers;

import java.util.HashSet;
import java.util.Set;

public class Simulation {
  public static int driveTillEqual(Driver... drivers) {
    int time;
    for (time = 0; notAllRumors(drivers) && time < 480; time++)
      driveAndGossip(drivers);
    return time;
  }

  private static void driveAndGossip(Driver[] drivers) {
    Set<Stop> stops = new HashSet<>();
    for (Driver d : drivers) {
      d.drive();
      stops.add(d.getStop());
    }
    for (Stop stop : stops)
      stop.gossip();
  }

  private static boolean notAllRumors(Driver[] drivers) {
    Set<Rumor> rumors = new HashSet<>();
    for (Driver d : drivers)
      rumors.addAll(d.getRumors());
    for (Driver d : drivers) {
      if (!d.getRumors().equals(rumors))
        return true;
    }
    return false;
  }
}
```

A quick perusal of this code will convince you that it is written in a very traditional OO style and that the objects encapsulate their own state relatively well.

### Clojure

When writing the Clojure version I did not start out with a design sketch. Rather, I depended upon my TDD tests to help me with the design. The tests are as follows:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch08_images.xhtml#f088-01)

```
(ns gossiping-bus-drivers-clojure.core-spec
  (:require [speclj.core :refer :all]
            [gossiping-bus-drivers-clojure.core :refer :all]))

(describe "gossiping bus drivers"
  (it "drives one bus at one stop"
    (let [driver (make-driver "d1" [:s1] #{:r1}4)
          world [driver]
          new-world (drive world)]
      (should= 1 (count new-world))
      (should= :s1 (-> new-world first :route first))))

  (it "drives one bus at two stops"
    (let [driver (make-driver "d1" [:s1 :s2] #{:r1})
          world [driver]
          new-world (drive world)]
      (should= 1 (count new-world))
      (should= :s2 (-> new-world first :route first))))

  (it "drives two buses at some stops"
    (let [d1 (make-driver "d1" [:s1 :s2] #{:r1})
          d2 (make-driver "d2" [:s1 :s3 :s2] #{:r2})
          world [d1 d2]
          new-1 (drive world)
          new-2 (drive new-1)]
      (should= 2 (count new-1))
      (should= :s2 (-> new-1 first :route first))
      (should= :s3 (-> new-1 second :route first))
      (should= 2 (count new-2))
      (should= :s1 (-> new-2 first :route first))
      (should= :s2 (-> new-2 second :route first))))

  (it "gets stops"
    (let [drivers #{{:name "d1" :route [:s1]}
                    {:name "d2" :route [:s1]}
                    {:name "d3" :route [:s2]}}]
      (should= {:s1 [{:name "d1" :route [:s1]}
                     {:name "d2" :route [:s1]}]
                :s2 [{:name "d3", :route [:s2]}]}
               (get-stops drivers)))
    )

  (it "merges rumors"
    (should= [{:name "d1" :rumors #{:r2 :r1}}
              {:name "d2" :rumors #{:r2 :r1}}]
             (merge-rumors [{:name "d1" :rumors #{:r1}}
                            {:name "d2" :rumors #{:r2}}])))


  (it "shares gossip when drivers are at same stop"
    (let [d1 (make-driver "d1" [:s1 :s2] #{:r1})
          d2 (make-driver "d2" [:s1 :s2] #{:r2})
          world [d1 d2]
          new-world (drive world)]
      (should= 2 (count new-world))
      (should= #{:r1 :r2} (-> new-world first :rumors))
      (should= #{:r1 :r2} (-> new-world second :rumors))))

  (it "passes acceptance test 1"
    (let [world [(make-driver "d1" [3 1 2 3] #{1})
                 (make-driver "d2" [3 2 3 1] #{2})
                 (make-driver "d3" [4 2 3 4 5] #{3})]]
      (should= 6 (drive-till-all-rumors-spread world))))

  (it "passes acceptance test 2"
    (let [world [(make-driver "d1" [2 1 2] #{1})
                 (make-driver "d2" [5 2 8] #{2})]]
          (should= :never (drive-till-all-rumors-spread world))))
  )
```

[4](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch08.xhtml#ch08fn4). `#{. . .}` represents a set in Clojure. A _set_ is a list of items that has no duplicates.

There are some interesting similarities between the Java tests and the Clojure tests. They are both quite wordy; although the Clojure tests contain half as many lines. The Java version has 12 tests whereas the Clojure version has only 8. This difference has a lot to do with the way the two different solutions were partitioned. The Clojure tests also play pretty fast and loose with the data.

Consider, for example, the `"merges rumors"` test. The `merge-rumors` function expects a list of drivers; however, the test does not create completely formed drivers. Rather, it creates abbreviated structures that look like drivers as far as the `merge-rumors` function is concerned.

The solution is all contained in a single, very short file:

[Click here to view code image](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch08_images.xhtml#f090-02)

```
(ns gossiping-bus-drivers-clojure.core
  (:require [clojure.set :as set]))

(defn make-driver [name route rumors]
  (assoc5 {} :name name :route (cycle6 route) :rumors rumors))

(defn move-driver [driver]
  (update7 driver :route rest))

(defn move-drivers [world]
  (map move-driver world))

(defn get-stops [world]
  (loop [world world
         stops {}]
    (if (empty? world)
      stops
      (let [driver (first world)
            stop (first (:route driver))
            stops (update stops stop conj driver)]
        (recur (rest world) stops)))))

(defn merge-rumors [drivers]
  (let [rumors (map :rumors drivers)
        all-rumors (apply set/union8 rumors)]
      (map #(assoc % :rumors all-rumors) drivers)))

(defn spread-rumors [world]
  (let [stops-with-drivers (get-stops world)
        drivers-by-stop (vals9 stops-with-drivers)]
    (flatten10 (map merge-rumors drivers-by-stop))))

(defn drive [world]
  (-> world move-drivers spread-rumors))

(defn drive-till-all-rumors-spread [world]
  (loop [world (drive world)
         time 1]
    (cond
      (> time 480) :never
      (apply = (map :rumors world)) time
      :else (recur (drive world) (inc time)))))
```

[5](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch08.xhtml#ch08fn5). `assoc` adds elements to a map. `(assoc {} :a 1)` returns `{:a 1}`.

[6](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch08.xhtml#ch08fn6). `cycle` returns a lazy (and “infinite”) list that simply repeats the input list endlessly. Thus, `(cycle [1 2 3])` returns `[1 2 3 1 2 3 1 2 3 …]`.

[7](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch08.xhtml#ch08fn7). The `update` function returns a new map with one element changed. `(update m k f a)` changes the `k` element of `m` by applying the function `(f e a)`, where `e` is the old value of element `k`. Thus, `(update {:x 1} :x inc)` returns `{:x 2}`.

[8](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch08.xhtml#ch08fn8). The `union` function is from the `set` namespace. Notice the `ns` at the top aliases the `clojure.set` namespace to just `set`.

[9](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch08.xhtml#ch08fn9). `vals` returns a list of all the values in a map. `keys` returns a list of all the keys in a map.

[10](https://learning.oreilly.com/library/view/functional-design-principles/9780138176518/ch08.xhtml#ch08fn10). The `flatten` function turns a list of lists into a list of all the elements. So `(flatten [[1 2][3 4]])` returns `[1 2 3 4]`.

This solution is 42 lines, whereas the Java solution is 145 lines spread among five files.

Both solutions have the concept of a Driver, but I made no attempt to encapsulate the concepts of Route, Stop, and Rumor into independent objects. They all just happily live within the Driver.

Worse, the Driver “object” is not an object in the traditional OO sense. It has no methods. There is one method in the system, `move-driver`, that operates on a single Driver, but it’s just a little helper function for the more interesting `move-drivers` function.

Six out of the eight functions take only the `world` as an argument. Thus, we might say that the only true object in this system is the `world`, and it has five methods. But even that is a stretch.

Even if we decide that the Driver is a kind of object, it is not mutable. The simulated `world` is nothing more than a list of immutable Drivers. The `drive` function accepts the `world` and produces a new `world` in which all the Drivers have been moved one step, and Rumors have been spread at any stop where more than one Driver has arrived.

That `drive` function is an example of an important concept. Notice how the `world` passes through a pipeline of functions. In this case there are only two, `move-drivers` and `spread-rumors`, but in larger systems the pipeline can be quite long. At each stage along that pipeline the `world` is modified into a slightly new form.

This tells us that the partitioning of this system is not about objects, but about functions. The relatively unpartitioned data passes from one independent function to the next.

You might argue that the Java code is relatively straightforward, whereas the Clojure code is too dense and obscure. Believe me when I say that it does not take very long to get comfortable with that density and that the perceived obscuration is an illusion based on unfamiliarity.

Is the lack of partitioning in the Clojure version a problem? Not at its current size; but if this program were to grow the way most systems grow, that problem would assert itself with a vengeance. Partitioning OO programs is a bit more natural than partitioning functional programs because the dividing lines are much more obvious and pronounced.

On the other hand, the dividing lines we chose for the Java version are not guaranteed to lead to an _effective_ partitioning. The warning is in the `drive` function of the Clojure program. It seems likely that a better partitioning of this system might lie along the different operations that manipulate the world, rather than things like Routes, Stops, and Rumors.

### Conclusion

We saw some differences in the Prime Factors and Bowling Game katas; but the differences were relatively minor. The differences in the Gossiping Bus Drivers kata were much more pronounced. This is likely because that last kata was a bit larger than the first two (I’d say twice the size), and also because it was a true finite state machine.

A _finite state machine_ moves from state to state, taking actions that depend upon the incoming events and the current state. When such systems are written in an OO style, the state tends to be stored in mutable objects that have dedicated methods. But in a functional style, the state remains externalized in immutable data structures that are passed through pipelines of functions.

We can perhaps conclude from this that programs that do simple calculations, like Prime Factors, are little affected by the OO or functional style. They are, after all, simple functions without any change of state. Programs in which state change is restricted to minor issues, such as array indexing, are only slightly affected by the difference in style. But those programs that are driven by changes of state from one moment to the next, like the Gossiping Bus Drivers program, will see profound differences between the two styles.

The OO style leads to a partitioning that is strongly related to data cohesion, whereas the functional style leads to a partitioning that is strongly related to behavioral cohesion. Which of these two is better is a question that I will leave for subsequent chapters.