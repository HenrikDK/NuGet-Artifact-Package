# Nuget Artifact Package
Purpose of this repository is to demonstrate a NuGet package setup that can be used by an organization to distribute commonly used artifacts files to a given .net project folder.

The intended use case is to allow distribution of static assets like internal configuration files, or gRPC client definition ".proto" files using an organizations internal nuget repository.

This use case is currently not supported by NuGet, since the .nuspec <contentFiles> tag only allows linking files as readonly resources into the project directory for ".Net Core" and > ".Net 5" projects.

## The Setup
The solution works by combining NuGet packaging and MS Build's more expanded functionality.

### NuGet is responsible for packaging 2 things:

  1. Any yaml files included in the project are included in .nupgk under a yamls folder as specified 
```
<file src="*.yaml" target="yamls" />
```

  2. A .props file that defines a MS Build step
```
<file src="build\*.props" target="build" />
```

### MS Build is responsible for:

  1. Once the package has been installed, on the next build MS Build will copy the nupkg's content into the project directory if the file doesn't already exist.
```
<Project>
    <Target Name="CopyFiles" BeforeTargets="Build">
        <ItemGroup>
            <File Include="$(MSBuildThisFileDirectory)..\yamls\*.*"></File>
        </ItemGroup>
        <Copy SourceFiles="@(File)" DestinationFolder="$(ProjectDir)"
              Condition="Exists('%(FullPath)')"/>
    </Target>
</Project>
```
Note that if you want to update the files on every build, you can remove the Condition attribute from the Copy tag.

## Limitations
This solution copies files on Build and not installation time, which will seem counter intuitive to users. 

The proper solution to this would be for NuGet to add support for what I've taken to calling "Nuget Artifact Packages". By adding a copyToProject attribute to the nuspec definition this process could be greatly simplified.

## Sources
This work is based on the work of number of sources: 

https://stackoverflow.com/questions/65218578/why-are-content-files-from-nuget-packages-added-as-links-in-net-core-projects-a

https://stackoverflow.com/questions/66696028/nuget-to-copy-files-into-the-project-directory

https://stackoverflow.com/questions/511829/msbuild-how-to-copy-files-that-may-or-may-not-exist

https://stackoverflow.com/questions/65218578/why-are-content-files-from-nuget-packages-added-as-links-in-net-core-projects-a