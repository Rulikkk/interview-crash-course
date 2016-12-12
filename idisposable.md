# IDisposable

`IDisposable` interface and unmanaged resources are very common interview questions.

## Requirements

There are only two situations when IDisposable does need to be

1. The class owns unmanaged resources.
2. The class owns managed \(IDisposable\) resources.

**IDisposable is not a destructor.** Remember that .NET has a garbage collector that works just fine without requiring you to set member variables to null.

Only classes that own resources should free them.  
 In particular, a class may have a reference to a shared resource: in this case, it should not free the resource because other classes may still be using it.

Rules from Stephen Cleary \([source](http://blog.stephencleary.com/2009/08/how-to-implement-idisposable-and.html)\):

1. Don’t do it \(unless you need to\).
2. For a class owning managed resources, implement IDisposable \(but not a finalizer\).
3. For a class owning a single unmanaged resource, implement both IDisposable and a finalizer
   .

## Reference implementation

```cs
using System;
using System.Collections.Generic;
using System.ComponentModel;
using System.Diagnostics;
using System.Runtime.InteropServices;
using System.Threading;

namespace IDisposableWorkShop
{
    // The example of class which is doesn't demand to implement IDisposable interface. Why?
    // The class doesn't own managed resources.
    // The class doesn't own unmanaged resources: 
    //    String is not Disposable and List is not Disposable 
    public sealed class ErrorList
    {
        private string category;
        private List<string> errors;

        public ErrorList(string category)
        {
            this.category = category;
            this.errors = new List<string>();
        }
    }

    // IDisposable only has one method: Dispose. This method has one important guarantee: 
    // it must be safe to call multiple times.
    // An implementation of Dispose may assume that it is not called from a finalizer thread,
    // that its instance is not being garbage collected, 
    // and that a constructor for its instance has completed successfully.
    // These assumptions makes it safe to access other managed objects.

    // Example of a correct IDisposable implementation.
    public sealed class SingleApplicationInstance : IDisposable
    {
        private Mutex namedMutex;
        private bool namedMutexCreatedNew;

        public SingleApplicationInstance(string applicationName)
        {
            this.namedMutex = new Mutex(false, applicationName, out namedMutexCreatedNew);
        }

        public bool AlreadyExisted
        {
            get { return !this.namedMutexCreatedNew; }
        }

        public void Dispose()
        {
            // This IDisposable.Dispose implementation is perfectly safe.
            // It can be safely called multiple times, because each of the IDisposable
            // implementations it invokes can be safely called multiple times.
            // This transitive property of IDisposable should be used to write 
            // simple Dispose implementations like this one.
            // There also can be cases with checking on null etc
            // (depends on how resource was created).
            namedMutex.Close();
        }

        // A class that owns a single unmanaged resource should not be responsible for 
        // anything else. It should only be responsible for closing that resource.

        // This is an example of a correct IDisposable implementation.
        // It is not ideal, however, because it does not inherit from SafeHandle
        public sealed class WindowStationHandle : IDisposable
        {
            public WindowStationHandle(IntPtr handle)
            {
                this.Handle = handle;
            }

            public WindowStationHandle()
                : this(IntPtr.Zero)
            {
            }

            public bool IsInvalid
            {
                get { return (this.Handle == IntPtr.Zero); }
            }

            public IntPtr Handle { get; set; }

            private void CloseHandle()
            {
                // Do nothing if the handle is invalid
                if (this.IsInvalid)
                {
                    // Being careful not to throw exceptions;
                    // CloseHandle may be called from the finalizer,
                    // and an exception at that point would crash the process. 
                    return;
                }

                // Close the handle, logging but otherwise ignoring errors
                if (!NativeMethods.CloseWindowStation(this.Handle))
                {
                    Trace.WriteLine("CloseWindowStation: " + new Win32Exception().Message);
                }

                // Set the handle to an invalid value
                this.Handle = IntPtr.Zero;
                // CloseHandle finishes by marking the handle as invalid, making it safe to
                // invoke this method multiple times;
                // this, in turn, makes it safe to invoke Dispose multiple times. It is
                // possible to move this “validity check” into the Dispose method,
                // but placing it in CloseHandle also allows invalid handles to be
                // passed into the constructor or set in the Handle property.
            }

            public void Dispose()
            {
                this.CloseHandle();
                // IDisposable.Dispose ends with a call to GC.SuppressFinalize(this).
                // This ensures that the object will remain live until after its
                // finalizer has been suppressed.
                GC.SuppressFinalize(this);
            }

            ~WindowStationHandle()
            {
                // If Dispose is not explicitly invoked, then the finalizer will eventually be invoked, which calls CloseHandle directly.
                this.CloseHandle();
            }
        }
        internal static partial class NativeMethods
        {
            [DllImport("user32.dll", SetLastError = true)]
            [return: MarshalAs(UnmanagedType.Bool)]
            internal static extern bool CloseWindowStation(IntPtr hWinSta);
        }

        // Below is presented implementation of IDisposable from MSDN
        // https://msdn.microsoft.com/ru-ru/library/system.idisposable(v=vs.110).aspx

        // If your app simply uses an object that implements T:System.IDisposable, don't provide
        // an T:System.IDisposable implementation. Instead, 
        // you should call the object's M:System.IDisposable.Dispose
        // implementation when you are finished using it.
        // Depending on your programming language, you can do this in one of two ways:
        //    By using a language construct such as the using statement in C# and Visual Basic.
        //    By wrapping the call to the M:System.IDisposable.Dispose implementation in a try/catch block.

        // The example of classical implementation of Dispose method
        class BaseClass : IDisposable
        {
            // Flag: Has Dispose already been called?
            bool disposed = false;

            // Public implementation of Dispose pattern callable by consumers.
            // Do not make this method virtual.
            // A derived class should not be able to override this method.
            public void Dispose()
            {
                Dispose(true);
                // You should call GC.SupressFinalize to take this object off the finalization queue
                // and prevent finalization code for this object from executing a second time.
                GC.SuppressFinalize(this);
            }

            // Protected implementation of Dispose pattern.
            // Dispose(bool disposing) executes in two distinct scenarios.
            // If disposing equals true, the method has been called directly
            // or indirectly by a user's code. Managed and unmanaged resources
            // can be disposed.
            // If disposing equals false, the method has been called by the
            // runtime from inside the finalizer and you should not reference
            // other objects. Only unmanaged resources can be disposed.
            protected virtual void Dispose(bool disposing)
            {
                // Check to see if Dispose has already been called.
                if (disposed)
                    return;

                // If disposing equals true, dispose all managed
                // and unmanaged resources.
                if (disposing)
                {
                    // Dispose managed resources.
                }

                // Call the appropriate methods to clean up
                // unmanaged resources here.
                // If disposing is false,
                // only the following code is executed.

                // Note disposing has been done.
                disposed = true;
            }

            // Use C# destructor syntax for finalization code.
            // This destructor will run only if the Dispose method
            // does not get called.
            // It gives your base class the opportunity to finalize.
            // Do not provide destructors in types derived from this class.
            ~BaseClass()
            {
                // Do not re-create Dispose clean-up code here.
                // Calling Dispose(false) is optimal in terms of
                // readability and maintainability.
                Dispose(false);
            }
        }
    }
}
```



