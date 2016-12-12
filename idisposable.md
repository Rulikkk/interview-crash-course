# IDisposable

`IDisposable` interface and unmanaged resources are very common interview questions.

## Requirements

There are only two situations when IDisposable does need to be 

1. The class owns unmanaged resources.
2. The class owns managed (IDisposable) resources.

**IDisposable is not a destructor.** Remember that .NET has a garbage collector that works just fine without requiring you to set member variables to null.

Only classes that own resources should free them. In particular, a class may have a reference to a shared resource: in this case, it should not free the resource because other classes may still be using it.

Rules from Stephen Cleary ([source](http://blog.stephencleary.com/2009/08/how-to-implement-idisposable-and.html)):
    
1. Don’t do it (unless you need to).
2. For a class owning managed resources, implement IDisposable (but not a finalizer).
3. For a class owning a single unmanaged resource, implement both IDisposable and a finalizer.


## Reference implementation

{% gist id="https://gist.github.com/Rulikkk/b251ad2aeecb397a33373be612ef3c51" %} {% endgist %}