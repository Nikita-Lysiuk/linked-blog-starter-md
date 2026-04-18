---
type: concept
title: "Observer Pattern"
created: 2026-04-18
updated: 2026-04-18
tags:
  - pattern
  - behavioral
  - observer
  - events
status: seed
complexity: intermediate
domain: software-architecture
related:
  - "[[Singleton]]"
  - "[[ECS]]"
  - "[[patterns/_index]]"
sources: []
---

# Observer Pattern

**Category:** Behavioral
**Intent:** Define a one-to-many dependency between objects so that when one object (Subject) changes state, all dependents (Observers) are notified and updated automatically.

---

## Structure

```
Subject (Publisher)
├── observers: List<IObserver>
├── Subscribe(observer)
├── Unsubscribe(observer)
└── Notify(event)

IObserver (Subscriber)
└── OnNotify(event)
```

---

## Pros

- Decouples Subject from Observers — Subject doesn't know what happens downstream
- Add/remove observers at runtime without changing Subject code
- Foundation for event-driven architecture and reactive systems
- Scales well: one event, N listeners (UI, audio, achievements, analytics all react to one health change)

## Cons

> [!contradiction] Memory Leaks via Forgotten Subscriptions
> In C#/Unity: if a listener object is destroyed without unsubscribing, the Subject holds a reference — preventing GC and causing null-ref exceptions. Always unsubscribe in OnDestroy.

- Unpredictable execution order when multiple observers listen to the same event
- Can create long, hard-to-trace "event chains" — debugging requires tracking all listeners
- C# `event` / delegates have overhead vs direct calls — profile in hot paths
- Observers can't easily return values to the Subject (use Command pattern for that)

## Game Context

**Use when:**
- UI reacts to game state: health bar updates when player health changes
- Achievement/analytics systems monitor game events without coupling to game logic
- Audio system plays sounds in response to gameplay events
- Multiplayer: local state changes broadcast to network layer

**Avoid when:**
- You need guaranteed execution order between observers
- The event carries data that observers must transform and return
- You're in a hot game loop path — profile first

**Unity implementation choices:**

| Approach | Coupling | Performance | Notes |
|----------|----------|-------------|-------|
| C# `event` + delegate | Low | Good | Standard, type-safe |
| UnityEvent | Low | OK | Inspector-wirable, serializable |
| ScriptableObject events | Very Low | Good | Ryan Hipple's architecture |
| [[ECS]] events | Minimal | Excellent | Structural changes as events |

---

## Code Skeleton (C#)

```csharp
// Generic typed event system
public interface IGameEvent<T>
{
    void Subscribe(Action<T> listener);
    void Unsubscribe(Action<T> listener);
    void Raise(T eventData);
}

public class GameEvent<T> : IGameEvent<T>
{
    private readonly List<Action<T>> _listeners = new();

    public void Subscribe(Action<T> listener)   => _listeners.Add(listener);
    public void Unsubscribe(Action<T> listener) => _listeners.Remove(listener);

    public void Raise(T eventData)
    {
        // Iterate copy to allow removal during iteration
        foreach (var listener in _listeners.ToArray())
            listener(eventData);
    }
}

// Usage — health system
public class PlayerHealth : MonoBehaviour
{
    public readonly GameEvent<int> OnHealthChanged = new();

    private int _health = 100;
    public void TakeDamage(int amount)
    {
        _health -= amount;
        OnHealthChanged.Raise(_health);
    }
}

// UI Listener — ALWAYS unsubscribe in OnDestroy
public class HealthBar : MonoBehaviour
{
    [SerializeField] private PlayerHealth _player;

    private void OnEnable()  => _player.OnHealthChanged.Subscribe(UpdateBar);
    private void OnDestroy() => _player.OnHealthChanged.Unsubscribe(UpdateBar);

    private void UpdateBar(int newHealth) { /* update slider */ }
}
```

```cpp
// C++ Observer with weak_ptr to avoid dangling references
#include <vector>
#include <functional>

template<typename T>
class Event {
public:
    using Handler = std::function<void(const T&)>;

    void Subscribe(Handler handler) {
        _handlers.push_back(std::move(handler));
    }

    void Raise(const T& data) {
        for (auto& handler : _handlers)
            handler(data);
    }

private:
    std::vector<Handler> _handlers;
};

// Usage
struct HealthChangedEvent { int newHealth; };
Event<HealthChangedEvent> onHealthChanged;

onHealthChanged.Subscribe([](const HealthChangedEvent& e) {
    // Update UI
});
onHealthChanged.Raise({75});
```

---

## Related Patterns

- **[[Singleton]]** — often combined as a global EventBus singleton
- **[[ECS]]** — ECS handles events via component addition/removal (structural changes)
- **Command** — for when observers need to return results or undo actions
- **Mediator** — centralizes all Subject-Observer relationships into one object
