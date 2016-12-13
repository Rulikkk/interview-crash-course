# IDisposable

`IDisposable` interface and unmanaged resources are very common interview questions.

## Rules from Stephen Cleary 
> http://blog.stephencleary.com/2009/08/how-to-implement-idisposable-and.html

This set of rules is better than MSDN and is very simple to understand. 

#### Rule 1: Don’t do it \(unless you need to\)

There are only two situations when IDisposable does need to be implemented:

- The class owns unmanaged resources.
- The class owns managed (IDisposable) resources.

_In all other cases, garbage collector will do its work without assistance._

#### Rule 2: For a class owning managed resources, implement IDisposable \(but not a finalizer\).

This implementation of IDisposable should only call Dispose for each owned resource. It should not have any other code: no “if” statements, no setting anything to null; just calls to Dispose or Close.

The class should not have a finalizer.

#### Rule 3: For a class owning a single unmanaged resource, implement both IDisposable and a finalizer

- A class that owns a single unmanaged resource should not be responsible for anything else. It should only be responsible for closing that resource.

- No class should be responsible for multiple unmanaged resources.

- No class should be responsible for both managed and unmanaged resources.

- This implementation of `IDisposable` should call an internal `CloseHandle` method and then end with a call to `GC.SuppressFinalize(this)`.

- The internal `CloseHandle` method should close the handle if it is a valid value, and then set the handle to an invalid value. This makes `CloseHandle` (and therefore Dispose) safe to call multiple times.

- The finalizer for the class should just call `CloseHandle`.

## References and implementation

For implementation, please, refer to Stephen Cleary's great blog entry, which contains all code samples: http://blog.stephencleary.com/2009/08/how-to-implement-idisposable-and.html

MSDN article on this topic, which is overcomplicated: https://msdn.microsoft.com/en-US/library/system.idisposable.aspx

Note: `IDisposable` is a very complicated topic, if you go into any detail of its implementation. Stephen's post provides all links for further research.