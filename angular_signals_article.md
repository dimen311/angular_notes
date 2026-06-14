# Angular Signals: How We Rewrote Our App and Why This Is the Future

I joined my current company about a year and a half ago, right when Angular Signals were just starting to gain real momentum. Massive credit goes to the creators of SolidJS—they were the ones who first popularized this concept, and the Angular team elegantly picked up the torch.

The core idea behind signals is actually much older. Reactive observable values existed in Knockout way back in 2010, and the term "signal" itself was solidified in the S.js library. In their Signals RFC, the Angular team explicitly mentioned they drew inspiration from Preact, Vue, and Solid.js.

And it completely fundamentally changed how our application works, making development so much easier. We started adopting them gradually: first, we built only new features using signals, and then we smoothly transitioned to refactoring our legacy code. Today, we have migrated almost entirely to signals.

---

## RxJS vs. Signals: What’s the Real Difference?

Previously, we heavily relied on the Observable + RxJS combo everywhere. The difference between the old approach and signals can be compared to a plumbing system vs. an Excel spreadsheet:

* **RxJS (Event Streams):** Think of it as a complex network of pipes. Data flows through them, and you have to subscribe at just the right time to catch the stream, all while constantly making sure nothing "bursts" (memory leaks).
* **Signals (State):** This is like a smart Excel cell. You define a formula, and if the data in the neighboring cells changes, your cell updates automatically. The value is always available to you, right here, right now.

To be fair, RxJS brought us reactivity a long time ago, but it came with a lot of boilerplate and ceremony: subscriptions, unsubscriptions, and dozens of operators. Signals provide that exact same reactivity but in a much simpler, synchronous wrapper. A signal always holds the latest value, which you can simply read without worrying about subscriptions or manual cleanup.

---

## Our Migration Journey

Because of this, we completely overhauled our architecture. Instead of building complex asynchronous chains of waits and locks, we simply fetch the data, show a loader, and relax with a cup of coffee while the UI reacts to the state change on its own.

By the way, Angular now features `resource()` and `httpResource()` specifically for this—they expose the loading state, error, and the actual data directly as a signal.

---

## What About Good Old RxJS?

Does our shift to signals mean RxJS should be completely written off? Absolutely not! RxJS is an incredibly powerful, mission-critical mechanism that remains irreplaceable for specific, complex tasks: handling high-frequency event streams, WebSocket connections, managing race conditions, debounce filtering, or orchestrating complex parallel async requests.

In software engineering, we rarely walk down just one path. Sometimes, you just need a smoothly paved road (and this is where simple, intuitive signals excel at state synchronization). Other times, you need to build complex suspension bridges out of RxJS operators to flexibly manage data streams. These tools weren't built to compete; they were designed to complement each other. But their architectural symbiosis is a story for another day.

---

## Breaking Old Taboos: Functions in Templates

Remember one of the most common interview questions? *"Why shouldn't you call functions in an Angular HTML template?"*

It used to be a strict taboo: every single Change Detection cycle would re-trigger the function, creating a massive performance bottleneck. We had to create extra variables in the component and bind them to the template, inflating our codebase.

With signals, this problem is gone. Calling `mySignal()` in a template is completely safe because it simply returns an already stored value in `O(1)` time. On top of that, a `computed` signal caches (memoizes) the result and only recalculates it when one of its dependencies actually changes. Reading a signal from a template is incredibly cheap, making everything lightning fast.

---

## Under the Hood

* **Glitch-Free Execution (Push/Pull Hybrid):** Angular signals operate on a hybrid model. When a signal changes, it doesn’t push new calculations instantly. Instead, it merely marks dependent signals as "dirty." The actual recalculation happens only when the value is explicitly requested (Pull). This eliminates UI flickering caused by intermediate states.
* **Dynamic Dependency Graph:** Signals rebuild their subscriptions during every execution cycle. If a `computed` signal contains an `if (condition()) { return data() }` statement, and `condition()` evaluates to `false`, the signal automatically unsubscribes from `data()`. No wasted CPU cycles.

---

## Goodbye, Default Change Detection

Despite all the magic of signals, here is my piece of advice: stop using the default change detection mechanism.

With the release of Angular v22, the Angular team officially reflected the true nature of `ChangeDetectionStrategy.Default` by renaming it to `Eager` and deprecating the old name. It’s a perfect description—it eagerly checks the entire component tree at the slightest hiccup.

Signals, on the other hand, provide **fine-grained reactivity** (surgically updating only the part of the DOM that changed). Stick with `OnPush` and move toward **Zoneless** applications, leaving eager checks in the past.

---

## The Ecosystem and a Few Tips

We’ve gained incredibly powerful tools like NgRx SignalStore and Signal Forms (which absolutely deserve a separate post). In the meantime, as you write code with signals, keep these two things in mind:

* **Use `untracked`:** When working with `effect()`, it’s easy to accidentally subscribe to a signal that shouldn’t trigger the effect again. Using `untracked()` lets you bypass those reads and removes unnecessary overhead.
* **Explore `linkedSignal`:** This feature is pure magic, allowing you to derive signals from existing ones while retaining the ability to update them manually.

Signals are undeniably the future.
