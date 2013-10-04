---
layout: post_page
title: Using Ironscheme with Unity
---

Ironscheme works with unity. First download Ironscheme from codeplex. Start IronScheme.Console-v2.exe (either directly or with mono) and run (compile-system-libraries). This is necessary because we need compiled dll's of the standard libraries to run in unity as a compiled build won't be able to read them from the scheme files. (At least not until I finish some tooling I'm working on). Once the files are done downloading copy all the .dll's (including IronScheme.dll) to a folder in the Assets directory of your unity project. You also must go into Edit -> Project Settings -> Player and under Other Settings / Optimization set Api compatibility level to .NET 2.0.

### Evaluating Scheme expressions from csharp

Evaluating scheme from a csharp script is easy.

{% highlight c# %}
using IronScheme;
IronScheme.RuntimeExtensions.Eval("(+ 1 2 3)");
{% endhighlight %}

Eval will evaluate any scheme expression so you could use that functionality to build a repl.

### Calling unity libraries from scheme.

Calling unity code from scheme is done using ironscheme's interop primitives. Their documentation can be found [here](http://ironscheme.codeplex.com/wikipage?title=clr-syntax&referringTitle=Documentation) For example, calling Debug.log would be done like this.

{% highlight scheme %}
(import (ironscheme clr))
(clr-using UnityEngine)
(clr-static-call Debug Log "log message")
{% endhighlight %}

You can not directly create a MonoBehavior class from ironscheme so using it in the unity framework is slightly different than using C# or Javascript. What  you can do is create ironscheme libraries that can be called from csharp and use that concept to create monobehaviors that call scheme code.

### Evaluating Scheme libraries from csharp

The best way I've found so far to write functionality in scheme and evaluate it on a gameobject is to create a scheme library and consume it from a monobehavior. Here's an example of how to do this. Create a Resources folder in your assets somewhere and make a scheme file in it called testlib.txt (yes txt) Then create a new c# script called TestLibrary.cs Here are their definitions.

testlib.txt
{% highlight scheme %}
(library (testlib)
(export update)
(import (rnrs)
        (ironscheme clr))
(clr-using UnityEngine)

(define (update)
  (clr-static-call Debug Log "testing"))
)
{% endhighlight %}

TestLibrary.cs
{% highlight c# %}
using UnityEngine;
using System.Collections;

using IronScheme;
using IronScheme.Runtime;

public class TestLibrary : MonoBehaviour {
	
  public Callable updateFn;
	
  void Start () {
    var script = (TextAsset)Resources.Load("testlib");
    IronScheme.RuntimeExtensions.Eval("(import (rnrs))");
    IronScheme.RuntimeExtensions.Eval(script.text);
    IronScheme.RuntimeExtensions.Eval("(import (testlib))");
    updateFn = IronScheme.RuntimeExtensions.Eval<Callable>("update");
  }
	
  void Update () {
    updateFn.Call();
  }
}
{% endhighlight %}


You can use this method to capture references to scheme code in csharp and call them. This is preferred over calling the functions directly with Eval for performance reasons. This works but isn't a great way to work with the scheme files. For one they are not exactly valid library files on their own and can't be loaded as a library from the repl. They are also hard to debug and don't work outside the context of the unity project because they reference the UnityEngine.dll. It does work however for evaluating scheme code in unity and is a good enough starting point.

### Next Steps

I'm working on some unity tooling that will better integrate scheme with the unity environment and will probably put it up on the unity asset store. Things like a repl built into the editor, scheme libraries that make it easier to work with unity objects, better ways to load user libraries than saving them as txt files and evaling them directly and some better ways to define unity components that can be attached to game objects and communicate with eachother. Coming Soon.

Huge thanks to [leppie](https://twitter.com/leppie) for helping me get this working.
