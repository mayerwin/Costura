## This is an add-in for [Fody](https://github.com/Fody/Fody/) 

Embeds dependencies as resources.

[Introduction to Fody](http://github.com/Fody/Fody/wiki/SampleUsage)

### Note: Costura VSIX

Users of the [Costura VSIX](http://visualstudiogallery.msdn.microsoft.com/70c0e54d-07de-4ff3-a62f-aac9c31e8c39) extension who are migrating to [Fody](https://github.com/Fody/fody) will want to use NuGet to Install the Costura.Fody package along with Fody itself to get the same functionality as before. This is because Fody is a general purpose weaver with plugins while Costura VSIX was specific to one scenario. See [Converting from Costura VSIX](https://github.com/Fody/Costura/wiki/ConvertingFromTheCosturaVSIX) for more information 


## Nuget package

Available here http://nuget.org/packages/Costura.Fody 

To Install from the Nuget Package Manager Console 
    
    PM> Install-Package Costura.Fody

# How it works

## Merge assemblies as embedded resources.

This approach uses a combination of two methods

 * Jeffrey Richter's suggestion of using [embedded resources as a method of merging assemblies](http://blogs.msdn.com/b/microsoft_press/archive/2010/02/03/jeffrey-richter-excerpt-2-from-clr-via-c-third-edition.aspx)
 * Einar Egilsson's suggestion [using cecil to create module initializers](http://tech.einaregilsson.com/2009/12/16/module-initializers-in-csharp/)

## Details 

This Task performs the following changes

 * Take all assemblies (and pdbs) that have been marked as "Copy Local" and embed them as resources in the target assembly.
 * Injects the following code into the module initializer of the target assembly. This code will be called when the assembly is loaded into memory

eg 

    static <Module>()
    {
        ILTemplate.Attach();
    }

 * Injects the following class into the target assembly. This means if an assembly load fails it will be loaded from the embedded resources.

  - [ILTemplate.cs](https://github.com/Fody/Costura/blob/master/Template/ILTemplate.cs)
  - [ILTemplateWithTempAssembly.cs](https://github.com/Fody/Costura/blob/master/Template/ILTemplateWithTempAssembly.cs)

# Configuration Options

All config options are access by modifying the `Costura` node in FodyWeavers.xml

## CreateTemporaryAssemblies

This will copy embedded files to disk before loading them into memory. This is helpful for some scenarios that expected an assembly to be loaded from a physical file.

*Defaults to `false`*

    <Costura CreateTemporaryAssemblies='true' />
    
## IncludeDebugSymbols

Controls if .pdbs for reference assemblies are also embedded.

*Defaults to `false`*

    <Costura IncludeDebugSymbols='false' />

## DisableCompression

Embedded assemblies are compressed by default, and uncompressed when they are loaded. You can turn compression off with this option.

*Defaults to `false`*

    <Costura DisableCompression='false' />
    
## ExcludeAssemblies

A list of assembly names to exclude from the default action of "embed all Copy Local references".

Do not include `.exe` or `.dll` in the names.

Can not be defined with `IncludeAssemblies`.

Can take two forms. 

As an element with items delimited by a newline.

    <Costura>
        <ExcludeAssemblies>
            Foo
            Bar
        </ExcludeAssemblies>
    </Costura>
    
Or as a attribute with items delimited by a pipe `|`.

    <Costura ExcludeAssemblies='Foo|Bar' />
    
        
## IncludeAssemblies

A list of assembly names to include from the default action of "embed all Copy Local references".

Do not include `.exe` or `.dll` in the names.

Can not be defined with `ExcludeAssemblies`.

Can take two forms. 

As an element with items delimited by a newline.

    <Costura>
        <IncludeAssemblies>
            Foo
            Bar
        </IncludeAssemblies>
    </Costura>
    
Or as a attribute with items delimited by a pipe `|`.

    <Costura IncludeAssemblies='Foo|Bar' />


## Unmanaged32Assemblies & Unmanaged64Assemblies

Mixed-mode assemblies cannot be loaded the same way as managed assemblies.

Therefore, to help Costura identify which assemblies are mixed-mode, and in what environment to load them in you should include their names in one or both of these lists.

Do not include `.exe` or `.dll` in the names.

Can take two forms. 

As an element with items delimited by a newline.

    <Costura>
        <Unmanaged32Assemblies>
            Foo32
            Bar32
        </Unmanaged32Assemblies>
        <Unmanaged64Assemblies>
            Foo64
            Bar64
        </Unmanaged64Assemblies>
    </Costura>
    
Or as a attribute with items delimited by a pipe `|`.

    <Costura 
        Unmanaged32Assemblies='Foo32|Bar32' 
        Unmanaged64Assemblies='Foo64|Bar64' />
