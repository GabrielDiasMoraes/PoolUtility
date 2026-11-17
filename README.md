# PoolUtility

A small object pool for Unity. It keeps deactivated instances of a prefab in a stack and reuses them, so you avoid repeated `Instantiate`/`Destroy` calls (and the GC spikes that come with them) for things like bullets, effects or UI items.

## Requirements

Unity 2021.3 (project created with 2021.3.5f1). The code only uses `UnityEngine` and the C# standard library.

## Install

Copy `PoolUtility/Assets/DailySolutions/PoolUtility` into your project's `Assets` folder. Namespace: `DailySolutions.PoolUtility.Scripts`.

## Usage

Make your prefab's script inherit from `PoolableItem`:

```csharp
using DailySolutions.PoolUtility.Scripts;

public class Bullet : PoolableItem
{
    public override void Init()
    {
        base.Init(); // activates the object and marks it as in use
        // reset your state here
    }
}
```

Create a pool and use it:

```csharp
var pool = new PoolUtility(bulletPrefab, amount: 20, parent: bulletsRoot);

Bullet bullet = pool.GetFromPool<Bullet>();   // reuses an instance or creates a new one
bullet.transform.position = muzzle.position;

pool.ReturnToPool(bullet);                    // or just call bullet.gameObject.SetActive(false)
```

Deactivating a pooled item (`SetActive(false)`, or disabling the component) also returns it to the pool through `PoolableItem.OnDisable`. If you override `OnDisable`, call `base.OnDisable()`.

## API

| Member | Description |
| --- | --- |
| `new PoolUtility(PoolableItem referenceItem, int amount = 0, Transform parent = null, Func<PoolableItem, Transform, PoolableItem> customInstantiator = null)` | Creates the pool and pre-instantiates `amount` inactive items. `customInstantiator` replaces `Object.Instantiate` (e.g. for a DI container). |
| `T GetFromPool<T>() where T : PoolableItem` | Pops an item (instantiating one if the pool is empty), calls `Init()` and re-parents it to `parent`. |
| `void ReturnToPool(PoolableItem item)` | Pushes the item back; ignored if it is already in the pool. |
| `PoolableItem.Init()` | Virtual. Activates the object and marks it as in use. |
| `PoolableItem.OnDisable()` | Protected virtual. Returns the item to its pool and deactivates it. |
| `IPoolableUtility` | Interface with `GetFromPool<T>` and `ReturnToPool`, for mocking or injection. |

## Status

Small personal utility from 2023, with no tests. The pool grows on demand and never shrinks or destroys items, and there is no maximum size. Not thread-safe (Unity main thread only).

## License

MIT, see [LICENSE](LICENSE).
