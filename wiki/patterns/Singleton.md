---
type: concept
title: "Singleton Pattern"
created: 2026-04-18
updated: 2026-04-18
tags:
  - pattern
  - creational
  - singleton
status: seed
complexity: basic
domain: software-architecture
related:
  - "[[Observer]]"
  - "[[patterns/_index]]"
sources: []
---

# Singleton Pattern

**Category:** Creational
**Intent:** Ensure a class has exactly one instance and provide a global access point to it.

---

## Structure

```
Singleton
├── static instance: Singleton
├── private constructor()
└── static GetInstance() → Singleton
```

---

## Pros

- Simple global access to shared systems (AudioManager, GameManager, InputSystem)
- Lazy initialization — instance created only when first accessed
- Thread-safe variant possible with double-checked locking
- Eliminates need to pass references down deep call chains

## Cons

> [!contradiction] Hidden Coupling
> Singletons create invisible dependencies. A class that calls `AudioManager.Instance.Play()` is tightly coupled to AudioManager — but this coupling doesn't appear in the class signature. Makes testing and refactoring harder.

- Hard to unit test — you can't inject a mock singleton
- Global mutable state: any system can corrupt it
- Initialization order issues across scenes (Unity)
- Violates Single Responsibility if the singleton accumulates logic
- Anti-pattern in ECS: use Systems + shared components instead

## Game Context

**Use when:**
- Truly global, single-instance services: `GameManager`, `AudioSystem`, `SceneLoader`, `InputHandler`
- You need scene-persistent objects in Unity and don't use a DI framework
- Prototyping — before architecture is settled

**Avoid when:**
- You want to unit test the consuming code
- Multiple configurations of the same system are needed (e.g., split-screen players)
- You're working in ECS — use World/System singletons (`SystemAPI.GetSingleton<T>()`) instead

**Unity note:** Use `DontDestroyOnLoad` + an `Awake` guard. Destroy duplicate instances on scene reload.

**Godot note:** Use Autoloads (Project Settings → Autoload) instead of manual singletons.

---

## Code Skeleton (C#)

```csharp
// Unity-compatible Singleton<T> base class
public class Singleton<T> : MonoBehaviour where T : MonoBehaviour
{
    private static T _instance;
    public static T Instance
    {
        get
        {
            if (_instance == null)
            {
                _instance = FindObjectOfType<T>();
                if (_instance == null)
                {
                    var go = new GameObject(typeof(T).Name);
                    _instance = go.AddComponent<T>();
                    DontDestroyOnLoad(go);
                }
            }
            return _instance;
        }
    }

    protected virtual void Awake()
    {
        if (_instance != null && _instance != this)
        {
            Destroy(gameObject);
            return;
        }
        _instance = this as T;
        DontDestroyOnLoad(gameObject);
    }
}

// Usage
public class AudioManager : Singleton<AudioManager>
{
    public void Play(AudioClip clip) { /* ... */ }
}

// Call site
AudioManager.Instance.Play(jumpSound);
```

```cpp
// C++ CRTP Singleton
template<typename T>
class Singleton {
public:
    static T& Get() {
        static T instance;
        return instance;
    }
    Singleton(const Singleton&) = delete;
    T& operator=(const T&) = delete;
protected:
    Singleton() = default;
};

class AudioManager : public Singleton<AudioManager> {
public:
    void Play(const std::string& clipName) { /* ... */ }
};

// Call site
AudioManager::Get().Play("jump");
```

---

## Related Patterns

- **Service Locator** — solves the same problem with more flexibility and testability
- **[[Observer]]** — often combined: Singleton EventBus pattern
- **[[ECS]]** — in ECS, replace singletons with singleton components + systems

> [!gap] No Service Locator page yet
> This vault doesn't have a Service Locator pattern page. File one when comparing approaches.
