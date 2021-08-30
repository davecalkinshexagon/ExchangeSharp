# ExchangeSharp
C# language binding for HOOPS Exchange. 

It can be built and used on the following platforms:
 - Windows
 - Ubuntu Linux
 - macOS

A video introducing ExchangeSharp can be found here: https://www.youtube.com/watch?v=C6XPWZAu1EE

## Expectations and Disclaimer
This is an unsupported implementation. It is being developed as experimental code. 

Don't read any further unless you are willing to work with the developers to identify issues that will arise.

If you are coming to this page with an expectation that you will obtain a complete and entirely tested C# wrapper, STOP! This is not the project for you.

Expect that you may need to modify delegate declarations in order to make some functionality accessible.

Expect an API that is very similar to the HOOPS Exchange native C API.

If you are going to use ExchangeSharp, you will need a basic understanding of the HOOPS Exchange C API.

## Description
This is a direct C# language binding for HOOPS Exchange implemented in the namespace `TS3D.Exchange.Direct`. 

> Functionality found in `A3DSDKDraw.h` is _not_ included.

ExchangeSharp was created using `libclang` to parse the Exchange headers. By traversing the declarations, we have reasonable certainty for complete coverage.

### What do you mean by "Direct"?
ExchangeSharp utilitizes the [P/Invoke](https://docs.microsoft.com/en-us/dotnet/standard/native-interop/pinvoke) approach for providing a cross-platform binding for the C# language. It does this by declaring all of the data types used by the native Exchange library in a compatible C# form.

"Direct" implies that the C# syntax you will use to access Exchange will closely resemble its C counterpart. Functions are looked up in the Exchange library and directly invoked.

ExchangeSharp is automatically generated by a libclang-based tool. As such, all source files (except Library.cs) should be considered disposable.

#### Disadvantages. 
I put this first because I don't think you'd read it if it came afer the advantages.

1. The generation algorithm isn't perfect (yet?). This means that some function signatures (see `API.cs`) are incorrect.
	1. This means you will have to make minor modifications to this file if you find an issue.
	1. Unless you want to make this modification every time you update the ExchangeSharp code base, you should provide the change so we can improve the generation algorithm.
1. Memory management within the Exchange data model must still be understood.
	1. The lifetime of C# objects in the Direct interface has no bearing on the underlying Exchange data model.
	1. You use ExchangeSharp functions to create objects, and you must use functions to explicitly delete them.
1. You must understand what an `IntPtr` is and how to use it.
	1. Exchange uses a lot of opaque pointers as "handles" into the Exchange data model. This is still the case even when using ExchangeSharp.

#### Advantages 
This approach has several advantages.

1. Memory: Data is not copied. It is directly mapped using `struct` objects. (See `ExchangeSharp/Direct/Structs.cs`)
1. Speed: There is no "middle" code. When you call an Exchange function using C#, you are directly invoking the code and nothing between. (See `ExchangeSharp/Direct/API.cs`)
1. Stability: Since it is based on the Exchange C API there will seldom be major changes to the interface.
1. Familiar: The code you write will look familiar:
```csharp
A3DAsmModelFileData d;
API.Initialize( out d );
if( A3DStatus.A3D_SUCCESS == API.A3DAsmModelFileGet( model_file, ref d ) ) {
    // Use object here

    // Free the object-
    API.A3DAsmModelFileGet( IntPtr.Zero, ref d );
}
```

#### Confidence

| API                        | Description                              | Confidence |
|----------------------------|------------------------------------------|------------|
| Libraray.cs                | Hand-written library management code     |     90%    |
| Structs.cs                 | Declarations of all struct data types    |     90%    |
| Enums.cs                   | Declarations of all enum data types      |     90%    |
| API.Initialize             | C# Implementation of A3D_INITIALIZE_DATA |     95%    |
| API.A3D*Get                | Get functions for structs                |     99%    |
| API.A3D*Create             | Create functions for structs             |     99%    |
| API.A3D*Edit               | Edit functions for structs               |     99%    |
| API.A3DAsmModelFileLoad*   | Loading functions                        |     90%    |
| API.A3DAsmModelFileExport* | Export functions                         |     90%    |
| API.cs (other)             | Other Exchange functions                 |     60%    |



## Prerequisites
In order to build and use ExchangeSharp, the following software components are required:
* [HOOPS Exchange 2020 SP2](https://developer.techsoft3d.com/hoops/exchange/downloads/latest/) (or binary compatible version)
	* Be sure you have Exchange installed and licensed correctly. 
	* Unpack the archive.
 	* Place `hoops_license.h` _and_ `hoops_license.cs` in the `include` folder.
	* Build and run the sample applications contained within.

* [.NET Core 3.1](https://dotnet.microsoft.com/download/dotnet-core/3.1)

## Building ExchangeSharp.dll
1. Clone the ExchangeSharp repository.
	* `git clone https://github.com/bflubacherts3d/ExchangeSharp.git`
1. Build ExchangeSharp
	* `dotnet build`
1. `ExchangeSharp.dll` can be found in the `bin/Debug/netcoreapp3.1` folder.

## Building and Running the Examples
The `examples` folder contains projects that illustrate basic use of ExchangeSharp. To build each project:
1. Copy `hoops_license.cs` to the example's folder you want to build.
1. Change your current working directory to the example's folder.
1. Use the command `dotnet build` to build
1. Run using `dotnet run`. Most examples take the command line argument `--exchange <exchange_bin_dir>`.
	1. Alternatively you can edit update your environment variable to include the Exchange bin folder for your platform (`PATH`, `LD_LIBRARY_PATH`, `DYLD_LIBRARY_PATH`).

# What is `Classes.cs`?
Similar to the [ExchangeToolkit](https://labs.techsoft3d.com/project/exchange-toolkit/), ExchangeSharp includes classes with the postfix "Wrapper". These classes call `API.Initialize` and `Get` in the constructor, and they free upon destruction. Using these classes results in code that looks like this:

```csharp
var d = new A3DAsmModelFileWrapper( model_file );
// Use object here
for( int idx = 0; idx < d.m_uiPOccurrencesSize; ++idx ) {
    var po = Marshal.ReadIntPtr(d.m_ppPOccurrences, idx * Marshal.SizeOf( typeof(IntPtr) ) );
    var po_d = new A3DAsmProductOccurrenceWrapper( po );
    // use Product Occurrence data...
}
```

Structs that do not have corresponding `Get` function (`A3DVector3dData` for example) are wrapped as well. Code provided in the `examples` folder make use of these wrapper classes in order to make the implementation more concise.



