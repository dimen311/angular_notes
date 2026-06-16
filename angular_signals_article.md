# Angular Signals: How We Rewrote Our App and Why This Is the Future

I joined my current company about a year and a half ago, right when Angular Signals were just starting to gain real momentum. Massive credit goes to the creators of SolidJS—they were the ones who first popularized this concept, and the Angular team elegantly picked up the torch.

The core idea behind signals is actually much older. Reactive observable values existed in Knockout way back in 2010, and the term "signal" itself was solidified in the S.js library. In their Signals RFC, the Angular team explicitly mentioned they drew inspiration from Preact, Vue, and Solid.js.

And it completely fundamentally changed how our application works, making development so much easier. We started adopting them gradually: first, we built only new features using signals, and then we smoothly transitioned to refactoring our legacy code. Today, we have migrated almost entirely to signals.

## RxJS vs. Signals: What’s the Real Difference?

Previously, we heavily relied on the Observable + RxJS combo everywhere. The difference between the old approach and signals can be compared to a plumbing system vs. an Excel spreadsheet:

- **RxJS (Event Streams):** Think of it as a complex network of pipes. Data flows through them, and you have to subscribe at just the right time to catch the stream, all while constantly making sure nothing "bursts" (memory leaks).
    
- **Signals (State):** This is like a smart Excel cell. You define a formula, and if the data in the neighboring cells changes, your cell updates automatically. The value is always available to you, right here, right now.
    

To be fair, RxJS brought us reactivity a long time ago, but it came with a lot of boilerplate and ceremony: subscriptions, unsubscriptions, and dozens of operators. Signals provide that exact same reactivity but in a much simpler, synchronous wrapper. A signal always holds the latest value, which you can simply read without worrying about subscriptions or manual cleanup.

## Our Migration Journey

Because of this, we completely overhauled our architecture. Instead of building complex asynchronous chains of waits and locks, we simply fetch the data, show a loader, and relax with a cup of coffee while the UI reacts to the state change on its own.

By the way, Angular now features `resource()` and `httpResource()` specifically for this—they expose the loading state, error, and the actual data directly as a signal.

## What About Good Old RxJS?

Does our shift to signals mean RxJS should be completely written off? Absolutely not! RxJS is an incredibly powerful, mission-critical mechanism that remains irreplaceable for specific, complex tasks: handling high-frequency event streams, WebSocket connections, managing race conditions, debounce filtering, or orchestrating complex parallel async requests.

In software engineering, we rarely walk down just one path. Sometimes, you just need a smoothly paved road (and this is where simple, intuitive signals excel at state synchronization). Other times, you need to build complex suspension bridges out of RxJS operators to flexibly manage data streams. These tools weren't built to compete; they were designed to complement each other. But their architectural symbiosis is a story for another day.

## Breaking Old Taboos: Functions in Templates

Remember one of the most common interview questions? _"Why shouldn't you call functions in an Angular HTML template?"_

It used to be a strict taboo: every single Change Detection cycle would re-trigger the function, creating a massive performance bottleneck. We had to create extra variables in the component and bind them to the template, inflating our codebase.

With signals, this problem is gone. Calling `mySignal()` in a template is completely safe because it simply returns an already stored value in O(1) time. On top of that, a `computed` signal caches (memoizes) the result and only recalculates it when one of its dependencies actually changes. Reading a signal from a template is incredibly cheap, making everything lightning fast.

## Under the Hood

- **Glitch-Free Execution (Push/Pull Hybrid):** Angular signals operate on a hybrid model. When a signal changes, it doesn’t push new calculations instantly. Instead, it merely marks dependent signals as "dirty." The actual recalculation happens only when the value is explicitly requested (Pull). This eliminates UI flickering caused by intermediate states.
    
- **Dynamic Dependency Graph:** Signals rebuild their subscriptions during every execution cycle. If a `computed` signal contains an `if (condition()) { return data() }` statement, and `condition()` evaluates to `false`, the signal automatically unsubscribes from `data()`. No wasted CPU cycles.
    

## Goodbye, Default Change Detection

Despite all the magic of signals, here is my piece of advice: stop using the legacy eager change detection mechanism.

With the release of Angular v22, the Angular team officially reflected the true nature of `ChangeDetectionStrategy.Default` by renaming it to `Eager` and deprecating the old name. It’s a perfect description—it eagerly checks the entire component tree at the slightest hiccup. Fortunately, **`OnPush` has officially become the default change detection strategy for new projects**, meaning newer setups get these performance benefits right out of the box.

Signals take this a step further by providing **fine-grained reactivity** (surgically updating only the part of the DOM that changed). 
This paves the way to confidently move toward **Zoneless** applications. Going Zoneless means removing `zone.js`, the library that historically intercepted every asynchronous event (Monkey patching): clicks, timers, HTTP requests, to force global re-checks. Removing `zone.js` also reduces bundle size( ~_30kB raw_). Signals explicitly tell Angular exactly _where_ and _when_ state has changed, giving us a much lighter, faster runtime without the global overhead.

## The Ecosystem and a Few Tips

We’ve gained incredibly powerful tools like `NgRx SignalStore` and `Signal Forms` (which absolutely deserve a separate post). In the meantime, as you write code with signals, keep these two tools in mind:

- **Use `untracked`:** This is essential when you need to read a value inside an `effect()` without establishing a dependency on it.
    
    > _Example:_ If you want to log an API call whenever `userName()` changes, but you also need to read a `sessionId()` signal _without_ having a session change re-trigger that log, wrap it in `untracked(() => sessionId())`.
``` javascript
effect(() => {
  const name = this.userName();
  untracked(() => {
    console.log(`User name changed to ${name}, session ID: ${sessionId()}`);
  });
});
```
    
    
- **Explore `linkedSignal`:** This solves the classic headache of local state that depends on a prop or upstream signal but must remain manually mutable by the user.
    
    > _Example:_ If you have a `shippingOptions` signal that should automatically reset to a default value whenever `shippingOptions()` changes, but still allows the user to manually select and override the selected option, `linkedSignal` handles this state synchronization flawlessly without nested, manual effects.
```javascript
import {
    Component,
    signal,
    linkedSignal
}
from '@angular/core';
 @Component({
    selector: 'app-shipping-picker',
    template: `
    <p>Selected: {{ selectedOption() }}</p> 
    @for (option of shippingOptions(); track option) 
    {
     <button (click)="changeShipping(option)"> {{ option }} </button>
    } 
    <button (click)="loadExpressOptions()">
      Load Express Options
    </button> `,
})
export class ShippingPickerComponent {
    shippingOptions = signal(['Ground', 'Air', 'Sea']);
     // Resets to first option whenever shippingOptions changes, 
     // but user can manually select any option 
     selectedOption = linkedSignal(() => this.shippingOptions()[0]); 
     // User manually selects an option from the component 
     changeShipping(option: string) { 
     this.selectedOption.set(option); 
     // manual override 
     } 
     // Simulates loading new options from an API 
     loadExpressOptions() { this.shippingOptions.set([
     'Carrier Pigeon (Weather Permitting)', 
     'Trained Falcon (Slightly Aggressive)', 
     'Yeeted via Catapult (For robust packages only. Parachute does not always open, backup available for an extra fee)', 
     'Delivered by Sonic the Hedgehog (If he takes damage, your package will scatter everywhere)']); 
     // selectedOption automatically resets to first one
      }
 }

```

## Tests!
And don't forget the tests, good old tests are still very much needed! Signals are completely synchronous, making testing an absolute breeze with zero async gymnastics required.

##### Signals are undeniably the future.
