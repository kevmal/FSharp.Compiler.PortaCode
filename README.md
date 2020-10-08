# FSharp.Compiler.PortaCode
The PortaCode F# code format and corresponding interpreter. 

* Used by Fabulous and others.

* Currently distributed by source inclusion or 'fslive' tool, no nuget package yet

* FsLive.Cli is a live programming "watch my project" command line tool which is experimental, e.g. 

       FsLive.Cli\bin\Debug\netcoreapp3.1\fslive.exe foo.fsx
       
* Wet paint, API will change

It's used for the "LiveUpdate" feature of Fabulous, to interpret the Elmish model/view/update application code on-device.

The interpreter may also be useful for other live checking tools, because you get escape the whole complication of actual IL generation, Reflection emit and reflection invoke, and no actual classes etc are generated. We can also adapt the interpreter over time to do things like report extra information back to the host.

### Code format

The input code format for the interpreter (PortaCode) is derived from FSharp.Compiler.Service expressions, the code is in this repo.

### Interpretation

The semantics of interpretation can differ from the semantics of .NET F# code. Perf is not good but in many live check scenarios you're sitting on a base set of DLLs which are regular .NET code and are efficiently invoked.

Library calls are implemented by reflection invoke.  It's the same interpreter we use on-device for Fabulous.

### Command line arguments

```
Usage: <tool> arg .. arg [-- <other-args>]
       <tool> @args.rsp  [-- <other-args>]
       <tool> ... Project.fsproj ... [-- <other-args>]

The default source is a single project file in the current directory.
The default output is a JSON dump of the PortaCode.

Arguments:
   --once            Don't enter watch mode (default: watch the source files of the project for changes)
   --send:<url>      Send the JSON-encoded contents of the PortaCode to the webhook
   --send            Equivalent to --send:http://localhost:9867/update
   --msbuildarg:arg  An MSBuild argument e.g. /p:Configuration=Release
   --dump            Dump the contents to console after each update
   --livecheck       Only evaluate those with a LiveCheck attribute. This uses on-demand execution semantics for top-level declarations
                     Also write an info file based on results of evaluation, and watch for .fsharp/foo.fsx.edit files and use the 
                     contents of those in preference to the source file
   <other-args>      All other args are assumed to be extra F# command line arguments, e.g. --define:FOO
```   

### LiveChecks

* A LiveCheck is a declaration like this: https://github.com/fsprojects/TensorFlow.FSharp/blob/master/examples/NeuralStyleTransfer-dsl.fsx#L109 …

* The attribute indicates the intent that that specific piece of code (and anything it depends on) should be run at development time. This is _not_ done by the IDE directly but by some other tool you have to start.

* An example tool is the "fslive.exe" tool from this repo here https://github.com/fsprojects/FSharp.Compiler.PortaCode/blob/master/src/ProcessCommandLine.fs#L46.  Like FsAutoComplete this watches for project changes and then recompiles using FCS and looks for LiveCheck attributes.  It then interprets those evaluations using reflection and collects information about the execution.  For example, it detects errors and detects when variables have been bound to particular values during interpretation.  The tool currently emits a ".fsharp/file.fsx.info" file containing extra information about the file "file.fsx" - extra error messages and extra tooltips.  An experimenta  FCS modification notices the existence of this file and incorporates the added information into Intellisense results. This keeps the checker tool totally decoupled from the IDE tooling.  

* This functionality may be reconfigured to be an [F# Analyzer](https://medium.com/lambda-factory/introducing-f-analyzers-772487889429).  
 

