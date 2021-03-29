<!-- .slide: data-background="#111111" -->

# Recap

<a href="https://coders.school">
    <img width="500" src="../coders_school_logo.png" alt="Coders School" class="plain">
</a>

___

### 1. What was the most surprising for you? 😲

### 2. What was the most obvious for you? 🥱

___
<!-- .slide: style="font-size: .81em" -->

## Points to remember

* <!-- .element: class="fragment fade-in" --> Always use <code>wait()</code> with a predicate!
* <!-- .element: class="fragment fade-in" --> Remember about spurious wake-ups and lost notifications
* <!-- .element: class="fragment fade-in" --> Remember about a fight for locking a mutex when you use <code>notify_all()</code>
* <!-- .element: class="fragment fade-in" --> You can't choose which thread should wake up when you use <code>notify_one()</code>

___
<!-- .slide: style="font-size: .65em" -->

## Pre-test 🤯

<div class="multicolumn">

<div style="width: 60%">

```cpp
// assume all necessary includes are here

int main() {
    std::mutex m;
    std::condition_variable cv;
    std::vector<int> v;
    std::vector<std::thread> producers;
    std::vector<std::thread> consumers;

    auto consume = [&] {
        std::unique_lock<std::mutex> ul(m);
        cv.wait(ul);
        std::cout << v.back();
        v.pop_back();
    };
    for (int i = 0; i < 10; i++) consumers.emplace_back(consume);

    auto produce = [&](int i) {
        {
            std::lock_guard<std::mutex> lg(m);
            v.push_back(i);
        }
        cv.notify_all();
    };
    for (int i = 0; i < 10; i++) producers.emplace_back(produce, i);

    for (auto && p : producers) p.join();
    for (auto && c : consumers) c.join();
}
```

</div>

<div class="col" style="margin-top: 20px">

1. <!-- .element: class="fragment highlight-green" --> there may be an Undefined Behavior in this code
2. <!-- .element: class="fragment highlight-red" --> the output is guaranteed to always be <code>0123456789</code>
3. <!-- .element: class="fragment highlight-red" --> <code>v</code> is always an empty vector at the end of this program
4. <!-- .element: class="fragment highlight-green" --> if some producers threads started before some consumers, we would have a deadlock because of lost notifications
5. <!-- .element: class="fragment highlight-green" --> a change from <code>notify_all()</code> to <code>notify_one()</code> guarantees that each consumer thread will receive a different number
6. <!-- .element: class="fragment highlight-green" --> this code can be improved by providing a predicate to <code>wait()</code> to disallow getting elements when the vector is empty

Note: 1, 4, 5, 6

</div>

</div>

___

## Post-work

* Ping-pong
  * difficult version - [homework/ping_pong.cpp](../homework/ping_pong.cpp)
  * easier version - [homework/ping_pong_easier.cpp](../homework/ping_pong_easier.cpp)

___

## Homework: ping-pong

<div class="multicolumn">

<div style="width: 60%; font-size: .9em;">

* <!-- .element: class="fragment fade-in" --> Thread A prints "ping" and the consecutive number
* <!-- .element: class="fragment fade-in" --> Thread B prints "pong" and the consecutive number
* <!-- .element: class="fragment fade-in" --> Ping always starts. Pong always ends.
* <!-- .element: class="fragment fade-in" --> Threads must work in turns. There may not be 2 consecutive pings or pongs. The program cannot end with a ping without a respective pong.
* <!-- .element: class="fragment fade-in" --> The program must be terminated either after the given number of repetitions or after the time limit, whichever occurs first. The reason for termination should be displayed on the screen.
* <!-- .element: class="fragment fade-in" --> Program parameters:
  * <!-- .element: class="fragment fade-in" --> number of repetitions
  * <!-- .element: class="fragment fade-in" --> time limit (in seconds)

</div>

<div style="width: 40%; font-size: .85em;">

```bash
$> g++ 03_ping_pong.cpp -lpthread
-std=c++17 -fsanitize=thread
$> ./a.out 1 10
ping 0
pong 0
Ping reached repetitions limit
Pong reached repetitions limit
$> ./a.out 12 1
ping 0
pong 0
ping 1
pong 1
ping 2
pong 2
Timeout
```

</div> <!-- .element: class="fragment fade-in" -->

</div>

___

## Tips

If you got stuck:

* <!-- .element: class="fragment fade-in" --> You need a mutex and a condition variable in your <code>PingPong</code> class
* <!-- .element: class="fragment fade-in" --> Wait for a condition variable with <code>wait_for()</code> in <code>stop()</code> function
* <!-- .element: class="fragment fade-in" --> Check the number of repetitions in ping and pong threads
* <!-- .element: class="fragment fade-in" --> Use an additional <code>std::atomic<bool></code> variable which will tell all threads to end, when the required conditions are met
* <!-- .element: class="fragment fade-in" --> Ping and pong threads should be using <code>wait()</code> to check if it's their turn to work. Use an additional variable that will be used in the predicate passed to <code>wait()</code>
* <!-- .element: class="fragment fade-in" --> The pong thread should end the program after reaching the repetition limit
