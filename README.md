# Extended filters for LeoEcsLite C# Entity Component System framework
Extended filters for [LeoECS Lite](https://github.com/Leopotam/ecslite).

> Tested on unity 2020.3 (not dependent on it) and contains assembly definition for compiling to separate assembly file for performance reason.

> **Important!** Don't forget to use `DEBUG` builds for development and `RELEASE` builds in production: all internal error checks / exception throwing works only in `DEBUG` builds and eleminated for performance reasons in `RELEASE`.

# Table of content
* [Socials](#socials)
* [Installation](#installation)
    * [As unity module](#as-unity-module)
    * [As source](#as-source)
* [Examples](#examples)
    * [Generic-constrained filters](#generic-constrained-filters)
    * [Custom-ordered filters](#custom-ordered-filters)
* [License](#license)

# Socials
[![discord](https://img.shields.io/discord/404358247621853185.svg?label=enter%20to%20discord%20server&style=for-the-badge&logo=discord)](https://discord.gg/5GZVde6)

# Installation

## As unity module
This repository can be installed as unity module directly from git url. In this way new line should be added to `Packages/manifest.json`:
```
"com.leopotam.ecslite.extendedfilters": "https://github.com/Leopotam/ecslite-extendedfilters.git",
```
By default last released version will be used. If you need trunk / developing version then `develop` name of branch should be added after hash:
```
"com.leopotam.ecslite.extendedfilters": "https://github.com/Leopotam/ecslite-extendedfilters.git#develop",
```

## As source
If you can't / don't want to use unity modules, code can be downloaded as sources archive from `Releases` page.

# Examples

## Generic-constrained filters
```csharp
class TestSystem : IEcsInitSystem, IEcsRunSystem {
    struct C1 { }
    struct C2 { }
    struct C3 { }

    EcsFilterExt<C1> _c1;
    EcsFilterExt<C1, C2> _c1c2;
    EcsFilterExt<C1, C2>.Exc<C3> _c1c2_c3;

    public void Init (EcsSystems systems) {
        var w = systems.GetWorld ();
        // Just call "Validate" method and filter will be initialized.
        _c1.Validate (w);
        _c1c2.Validate (w);
        _c1c2_c3.Validate (w);
    }

    public void Run (EcsSystems systems) {
        // We can iterate over extended filter as usual.
        foreach (var entity in _c1.Filter ()) {
            ref var c1 = ref _c1.Inc1 ().Get (entity);
        }
        // We can request any pool from filter constraints.
        foreach (var entity in _c1c2.Filter ()) {
            ref var c1 = ref _c1c2.Inc1 ().Get (entity);
            ref var c2 = ref _c1c2.Inc2 ().Get (entity);
        }
    }
}
```
> **Important!** `EcsFilterExt` supports up to 8 include constraints and up to 4 exclude constraints.

## Custom-ordered filters
```csharp
sealed class TestSystem : IEcsInitSystem, IEcsRunSystem {
    struct C1 {
        public int Value;
    }

    EcsFilterExt<C1> _filter;
    EcsFilterReorderHandler _cb;

    public void Init (EcsSystems systems) {
        var w = systems.GetWorld ();
        _filter.Validate (w);
        for (var i = 0; i < 3; i++) {
            var entity = w.NewEntity ();
            _filter.Inc1 ().Add (entity).Value = i;
        }
        // we want to sort entities based on C1.Value data.
        _cb = entity => _filter.Inc1 ().Get (entity).Value;
    }

    public void Run (EcsSystems systems) {
        // reordering, entities will be resorted.
        _filter.Filter ().Reorder (_cb);
        
        // change entity cost to opposite value for test.
        foreach (var entity in _filter.Filter ()) {
            _filter.Inc1 ().Get (entity).Value *= -1;
        }
    }
}
```
> **Important!** `EcsFilter.Reorder()` is not thread safe!

# License
The software is released under the terms of the [MIT license](./LICENSE.md).

No personal support or any guarantees.
