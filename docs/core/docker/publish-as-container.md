---
title: Containerize an app with dotnet publish
description: In this tutorial, you'll learn how to containerize a .NET application with dotnet publish.
ms.date: 12/18/2023
ms.topic: tutorial
zone_pivot_groups: dotnet-version-7-8
---

# Containerize a .NET app with dotnet publish

Containers have many features and benefits, such as being an immutable infrastructure, providing a portable architecture, and enabling scalability. The image can be used to create containers for your local development environment, private cloud, or public cloud. In this tutorial, you'll learn how to containerize a .NET application using the [dotnet publish](../tools/dotnet-publish.md) command.

## Prerequisites

Install the following prerequisites:

:::zone pivot="dotnet-8-0"

- [.NET 8+ SDK](https://dotnet.microsoft.com/download/dotnet/8.0)\
If you have .NET installed, use the `dotnet --info` command to determine which SDK you're using.
- [Docker Community Edition](https://www.docker.com/products/docker-desktop)

:::zone-end
:::zone pivot="dotnet-7-0"

- [.NET 7+ SDK](https://dotnet.microsoft.com/download/dotnet/7.0)\
If you have .NET installed, use the `dotnet --info` command to determine which SDK you're using.
- [Docker Community Edition](https://www.docker.com/products/docker-desktop)

:::zone-end

In addition to these prerequisites, it's recommended that you're familiar with [Worker Services in .NET](../extensions/workers.md).

## Create .NET app

You need a .NET app to containerize, so start by creating a new app from a template. Open your terminal, create a working folder (*sample-directory*) if you haven't already, and change directories so that you're in it. In the working folder, run the following command to create a new project in a subdirectory named *Worker*:

```dotnetcli
dotnet new worker -o Worker -n DotNet.ContainerImage
```

Your folder tree looks like the following:

```Directory
📁 sample-directory
    └──📂 Worker
        ├──appsettings.Development.json
        ├──appsettings.json
        ├──DotNet.ContainerImage.csproj
        ├──Program.cs
        ├──Worker.cs
        └──📂 obj
            ├── DotNet.ContainerImage.csproj.nuget.dgspec.json
            ├── DotNet.ContainerImage.csproj.nuget.g.props
            ├── DotNet.ContainerImage.csproj.nuget.g.targets
            ├── project.assets.json
            └── project.nuget.cache
```

The `dotnet new` command creates a new folder named _Worker_ and generates a worker service that, when run, logs a message every second. From your terminal session, change directories and navigate into the *Worker* folder. Use the `dotnet run` command to start the app.

```dotnetcli
dotnet run
Building...
info: DotNet.ContainerImage.Worker[0]
      Worker running at: 10/18/2022 08:56:00 -05:00
info: Microsoft.Hosting.Lifetime[0]
      Application started. Press Ctrl+C to shut down.
info: Microsoft.Hosting.Lifetime[0]
      Hosting environment: Development
info: Microsoft.Hosting.Lifetime[0]
      Content root path: .\Worker
info: DotNet.ContainerImage.Worker[0]
      Worker running at: 10/18/2022 08:56:01 -05:00
info: DotNet.ContainerImage.Worker[0]
      Worker running at: 10/18/2022 08:56:02 -05:00
info: DotNet.ContainerImage.Worker[0]
      Worker running at: 10/18/2022 08:56:03 -05:00
info: Microsoft.Hosting.Lifetime[0]
      Application is shutting down...
Attempting to cancel the build...
```

The worker template loops indefinitely. Use the cancel command <kbd>Ctrl+C</kbd> to stop it.

:::zone pivot="dotnet-7-0"

## Add NuGet package

The [Microsoft.NET.Build.Containers](https://www.nuget.org/packages/Microsoft.NET.Build.Containers) NuGet package is currently required to publish non-web projects as a container. To add the `Microsoft.NET.Build.Containers` NuGet package to the worker template, run the following [dotnet add package](../tools/dotnet-add-package.md) command:

```dotnetcli
dotnet add package Microsoft.NET.Build.Containers
```

> [!TIP]
> If you're building a web app and using .NET SDK 7.0.300 or later, then the package isn't required&mdash;the SDK contains the same functionality out of the box.

:::zone-end

## Set the container image name

There are various configuration options available when publishing an app as a container.

:::zone pivot="dotnet-8-0"

By default, the container image name is the `AssemblyName` of the project. If that name is invalid as a container image name, you can override it by specifying a `ContainerRepository` as shown in the following project file:

:::code language="xml" source="snippets/8.0/Worker/DotNet.ContainerImage.csproj" highlight="8":::

For more information, see [ContainerRepository](#containerrepository).

:::zone-end
:::zone pivot="dotnet-7-0"

By default, the container image name is the `AssemblyName` of the project. If that name is invalid as a container image name, you can override it by specifying a (`ContainerImageName` obsolete) or the preferred `ContainerRepository` as shown in the following project file:

:::code language="xml" source="snippets/7.0/Worker/DotNet.ContainerImage.csproj" highlight="8":::

For more information, see [ContainerImageName](#containerimagename).

:::zone-end

## Publish .NET app

To publish the .NET app as a container, use the following [dotnet publish](../tools/dotnet-publish.md) command:

```dotnetcli
dotnet publish --os linux --arch x64 /t:PublishContainer -c Release
```

The preceding .NET CLI command publishes the app as a container:

- Targeting Linux as the OS (`--os linux`).
- Specifying an x64 architecture (`--arch x64`).
- Using the release configuration (`-c Release`).

> [!IMPORTANT]
> To build the container locally, you must have the Docker daemon running. If it isn't running when you attempt to publish the app as a container, you'll experience an error similar to the following:
>
> ```console
> ..\build\Microsoft.NET.Build.Containers.targets(66,9): error MSB4018:
>    The "CreateNewImage" task failed unexpectedly. [..\Worker\DotNet.ContainerImage.csproj]
> ```

> [!TIP]
> Depending on the type of app you're containerizing, the command-line switches (options) might vary. For example, the `/t:PublishContainer` argument is only required for non-web .NET apps, such as `console` and `worker` templates. For web templates, replace the `/t:PublishContainer` argument with `-p:PublishProfile=DefaultContainer`. For more information, see [.NET SDK container builds, issue #141](https://github.com/dotnet/sdk-container-builds/issues/141).

The command produces output similar to the example output:

:::zone pivot="dotnet-8-0"

```dotnetcli
Determining projects to restore...
  All projects are up-to-date for restore.
  DotNet.ContainerImage -> .\Worker\bin\Release\net8.0\linux-x64\DotNet.ContainerImage.dll
  DotNet.ContainerImage -> .\Worker\bin\Release\net8.0\linux-x64\publish\
  Building image 'dotnet-worker-image' with tags latest on top of base image mcr.microsoft.com/dotnet/aspnet:8.0
  Pushed container 'dotnet-worker-image:latest' to Docker daemon
```

:::zone-end
:::zone pivot="dotnet-7-0"

```dotnetcli
Determining projects to restore...
  All projects are up-to-date for restore.
  DotNet.ContainerImage -> .\Worker\bin\Release\net7.0\linux-x64\DotNet.ContainerImage.dll
  DotNet.ContainerImage -> .\Worker\bin\Release\net7.0\linux-x64\publish\
  Building image 'dotnet-worker-image' with tags 1.0.0 on top of base image mcr.microsoft.com/dotnet/aspnet:7.0
  Pushed container 'dotnet-worker-image:1.0.0' to Docker daemon
```

:::zone-end

This command compiles your worker app to the *publish* folder and pushes the container to your local docker registry.

## Configure container image

You can control many aspects of the generated container through MSBuild properties. In general, if you can use a command in a _Dockerfile_ to set some configuration, you can do the same via MSBuild.

> [!NOTE]
> The only exceptions to this are `RUN` commands. Due to the way containers are built, those cannot be emulated. If you need this functionality, you'll need to use a _Dockerfile_ to build your container images.

### `ContainerBaseImage`

The container base image property controls the image used as the basis for your image. By default, the following values are inferred based on the properties of your project:

- If your project is self-contained, the `mcr.microsoft.com/dotnet/runtime-deps` image is used as the base image.
- If your project is an ASP.NET Core project, the `mcr.microsoft.com/dotnet/aspnet` image is used as the base image.
- Otherwise the `mcr.microsoft.com/dotnet/runtime` image is used as the base image.

The tag of the image is inferred to be the numeric component of your chosen `TargetFramework`. For example, a project targeting `net6.0` results in the `6.0` tag of the inferred base image, and a `net7.0-linux` project uses the `7.0` tag, and so on.

If you set a value here, you should set the fully qualified name of the image to use as the base, including any tag you prefer:

:::zone pivot="dotnet-8-0"

```xml
<PropertyGroup>
    <ContainerBaseImage>mcr.microsoft.com/dotnet/runtime:8.0</ContainerBaseImage>
</PropertyGroup>
```

:::zone-end
:::zone pivot="dotnet-7-0"

```xml
<PropertyGroup>
    <ContainerBaseImage>mcr.microsoft.com/dotnet/runtime:7.0</ContainerBaseImage>
</PropertyGroup>
```

:::zone-end

:::zone pivot="dotnet-8-0"

### ContainerFamily

Starting with .NET 8, you can use the `ContainerFamily` MSBuild property to choose a different family of Microsoft-provided container images as the base image for your app. When set, this value is appended to the end of the selected TFM-specific tag, changing the tag provided. For example, to use the Alpine Linux variants of the .NET base images, you can set `ContainerFamily` to `alpine`:

```xml
<PropertyGroup>
    <ContainerFamily>alpine</ContainerFamily>
</PropertyGroup>
```

The preceding project configuration results in a final tag of `8.0-alpine` for a .NET 8-targeting app.

This field is free-form, and often can be used to select different operating system distributions, default package configurations, or any other _flavor_ of changes to a base image. This field is ignored when `ContainerBaseImage` is set. For more information, see [.NET container images](container-images.md).

:::zone-end

### `ContainerRuntimeIdentifier`

The container runtime identifier property controls the operating system and architecture used by your container if your [ContainerBaseImage](#containerbaseimage) supports more than one platform. For example, the `mcr.microsoft.com/dotnet/runtime` image currently supports `linux-x64`, `linux-arm`, `linux-arm64` and `win10-x64` images all behind the same tag, so the tooling needs a way to be told which of these versions you intend to use. By default, this is set to the value of the `RuntimeIdentifier` that you chose when you published the container. This property rarely needs to be set explicitly - instead use the `-r` option to the `dotnet publish` command. If the image you've chosen doesn't support the `RuntimeIdentifier` you've chosen, results in an error that describes the RuntimeIdentifiers the image does support.

You can always set the `ContainerBaseImage` property to a fully qualified image name, including the tag, to avoid needing to use this property at all.

```xml
<PropertyGroup>
    <ContainerRuntimeIdentifier>linux-arm64</ContainerRuntimeIdentifier>
</PropertyGroup>
```

For more information regarding the runtime identifiers supported by .NET, see [RID catalog](../rid-catalog.md).

### `ContainerRegistry`

The container registry property controls the destination registry, the place that the newly created image will be pushed to. Be default it's pushed to the local Docker daemon, but you can also specify a remote registry.  When using a remote registry that requires authentication, you authenticate using the well-known `docker login` mechanisms. For more information, See [authenticating to container registries](https://aka.ms/dotnet/containers/auth) for more details. For a concrete example of using this property, consider the following XML example:

```xml
<PropertyGroup>
    <ContainerRegistry>registry.mycorp.com:1234</ContainerRegistry>
</PropertyGroup>
```

This tooling supports publishing to any registry that supports the [Docker Registry HTTP API V2](https://docs.docker.com/registry/spec/api/).  This includes the following registries explicitly (and likely many more implicitly):

* [Azure Container Registry](https://azure.microsoft.com/products/container-registry)
* [Amazon Elastic Container Registry](https://aws.amazon.com/ecr/)
* [Google Artifact Registry](https://cloud.google.com/artifact-registry)
* [Docker Hub](https://hub.docker.com/)
* [GitHub Packages](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-container-registry)
* [GitLab-hosted Container Registry](https://docs.gitlab.com/ee/user/packages/container_registry/)
* [Quay.io](https://quay.io/)

For notes on working with these registries, see the [registry-specific notes](https://aka.ms/dotnet/containers/auth#notes-for-specific-registries).

:::zone pivot="dotnet-8-0"

### `ContainerRepository`

The container repository is the name of the image itself, for example, `dotnet/runtime` or `my-app`. By default, the `AssemblyName` of the project is used.

```xml
<PropertyGroup>
    <ContainerRepository>my-app</ContainerRepository>
</PropertyGroup>
```

:::zone-end
:::zone pivot="dotnet-7-0"

### `ContainerImageName`

The container image name controls the name of the image itself, for example, `dotnet/runtime` or `my-app`. By default, the `AssemblyName` of the project is used.

```xml
<PropertyGroup>
    <ContainerImageName>my-app</ContainerImageName>
</PropertyGroup>
```

> [!NOTE]
> Starting with .NET 8, `ContainerImageName` is deprecated in favor of `ContainerRepository`.

:::zone-end

Image names consist of one or more slash-delimited segments, each of which can only contain lowercase alphanumeric characters, periods, underscores, and dashes, and must start with a letter or number. Any other characters result in an error being thrown.

### `ContainerImageTag(s)`

The container image tag property controls the tags that are generated for the image. To specify a single tag use `ContainerImageTag` and for multiple tags use `ContainerImageTags`. Tags are often used to refer to different versions of an app, but they can also refer to different operating system distributions, or even different configurations.

:::zone pivot="dotnet-8-0"

Starting with .NET 8, when a tag isn't provided the default is `latest`.

:::zone-end
:::zone pivot="dotnet-7-0"

By default, the `Version` of the project is used as the tag value.

:::zone-end

To override the default, specify either of the following:

```xml
<PropertyGroup>
    <ContainerImageTag>1.2.3-alpha2</ContainerImageTag>
</PropertyGroup>
```

To specify multiple tags, use a semicolon-delimited set of tags in the `ContainerImageTags` property, similar to setting multiple `TargetFrameworks`:

```xml
<PropertyGroup>
    <ContainerImageTags>1.2.3-alpha2;latest</ContainerImageTags>
</PropertyGroup>
```

Tags can only contain up to 127 alphanumeric characters, periods, underscores, and dashes. They must start with an alphanumeric character or an underscore. Any other form results in an error being thrown.

> [!NOTE]
> When using `ContainerImageTags`, the tags are delimited by a `;` character. If you're calling `dotnet publish` from the command line (as is the case with most CI/CD environments), you'll need to outer wrap the values in a single `'` and inner wrap with double quotes `"`, for example (`='"tag-1;tag-2"'`). Consider the following `dotnet publish` command:
>
> ```dotnetcli
> dotnet publish -p ContainerImageTags='"1.2.3-alpha2;latest"'
> ```

### `ContainerLabel`

The container label adds a metadata label to the container. Labels have no impact on the container at run time, but are often used to store version and authoring metadata for use by security scanners and other infrastructure tools. You can specify any number of container labels.

The `ContainerLabel` node has two attributes:

- `Include`: The key of the label.
- `Value`: The value of the label (this may be empty).

```xml
<ItemGroup>
    <ContainerLabel Include="org.contoso.businessunit" Value="contoso-university" />
</ItemGroup>
```

For a list of labels that are created by default, see [default container labels](#default-container-labels).

## Configure container execution

To control the execution of the container, you can use the following MSBuild properties.

### `ContainerWorkingDirectory`

The container working directory node controls the working directory of the container, the directory that commands are executed within if not other command is run.

By default, the `/app` directory value is used as the working directory.

```xml
<PropertyGroup>
    <ContainerWorkingDirectory>/bin</ContainerWorkingDirectory>
</PropertyGroup>
```

### `ContainerPort`

The container port adds TCP or UDP ports to the list of known ports for the container. This enables container runtimes like Docker to map these ports to the host machine automatically. This is often used as documentation for the container, but can also be used to enable automatic port mapping.

The `ContainerPort` node has two attributes:

- `Include`: The port number to expose.
- `Type`: Defaults to `tcp`, valid values are either `tcp` or `udp`.

```xml
<ItemGroup>
    <ContainerPort Include="80" Type="tcp" />
</ItemGroup>
```

:::zone pivot="dotnet-8-0"

Starting with .NET 8, the `ContainerPort` is inferred when not explicitly provided based on several well-known ASP.NET environment variables:

- `ASPNETCORE_URLS`
- `ASPNETCORE_HTTP_PORTS`
- `ASPNETCORE_HTTPS_PORTS`

If these environment variables are present, their values are parsed and converted to TCP port mappings. These environment variables are read from your base image, if present, or from the environment variables defined in your project through `ContainerEnvironmentVariable` items. For more information, see [ContainerEnvironmentVariable](#containerenvironmentvariable).

:::zone-end

### `ContainerEnvironmentVariable`

The container environment variable node allows you to add environment variables to the container. Environment variables are accessible to the app running in the container immediately, and are often used to change the run-time behavior of the running app.

The `ContainerEnvironmentVariable` node has two attributes:

- `Include`: The name of the environment variable.
- `Value`: The value of the environment variable.

```xml
<ItemGroup>
  <ContainerEnvironmentVariable Include="LOGGER_VERBOSITY" Value="Trace" />
</ItemGroup>
```

For more information, see [.NET environment variables](../tools/dotnet-environment-variables.md).

## Configure container commands

By default, the container tools launch your app using either the generated AppHost binary for your app (if your app uses an AppHost), or the `dotnet` command plus your app's DLL.

:::zone pivot="dotnet-8-0"

However, you can control how your app is executed by using some combination of `ContainerAppCommand`, `ContainerAppCommandArgs`, `ContainerDefaultArgs`, and `ContainerAppCommandInstruction`.

These different configuration points exist because different base images use different combinations of the container `ENTRYPOINT` and `COMMAND` properties, and you want to be able to support all of them. The defaults should be useable for most apps, but if you want to customize your app launch behavior you should:

- Identify the binary to run and set it as `ContainerAppCommand`
- Identify which arguments are _required_ for your application to run and set them as `ContainerAppCommandArgs`
- Identify which arguments (if any) are _optional_ and could be overridden by a user and set them as `ContainerDefaultArgs`
- Set `ContainerAppCommandInstruction` to `DefaultArgs`

For more information, see the following configuration items.

### `ContainerAppCommand`

The app command configuration item is the logical entry point of your app. For most apps, this is the AppHost, the generated executable binary for your app. If your app doesn't generate an AppHost, then this command will typically be `dotnet <your project dll>`. These values are applied after any `ENTRYPOINT` in your base container, or directly if no `ENTRYPOINT` is defined.

The `ContainerAppCommand` configuration has a single `Include` property, which represents the command, option, or argument to use in the entrypoint command:

```xml
<ItemGroup Label="ContainerAppCommand Assignment">
  <!-- This is how you would start the dotnet ef tool in your container -->
  <ContainerAppCommand Include="dotnet" />
  <ContainerAppCommand Include="ef" />

  <!-- This shorthand syntax means the same thing, note the semicolon separating the tokens. -->
  <ContainerAppCommand Include="dotnet;ef" />
</ItemGroup>
```

### `ContainerAppCommandArgs`

This app command args configuration item represents any logically required arguments for your app that should be applied to the `ContainerAppCommand`. By default, none are generated for an app. When present, the args are applied to your container when it's run.

The `ContainerAppCommandArgs` configuration has a single `Include` property, which represents the option or argument to apply to the `ContainerAppCommand` command.

```xml
<ItemGroup>
  <!-- Assuming the ContainerAppCommand defined above, 
       this would be the way to force the database to update.
  -->
  <ContainerAppCommandArgs Include="database" />
  <ContainerAppCommandArgs Include="update" />

  <!-- This is the shorthand syntax for the same idea -->
  <ContainerAppCommandArgs Include="database;update" />
</ItemGroup>
```

### `ContainerDefaultArgs`

This default args configuration item represents any user-overridable arguments for your app. This is a good way to provide defaults that your app might need to run in a way that makes it easy to start, yet still easy to customize.

The `ContainerDefaultArgs` configuration has a single `Include` property, which represents the option or argument to apply to the `ContainerAppCommand` command.

```xml
<ItemGroup>
  <!-- Assuming the ContainerAppCommand defined above, 
       this would be the way to force the database to update.
  -->
  <ContainerDefaultArgs Include="database" />
  <ContainerDefaultArgs Include="update" />

  <!-- This is the shorthand syntax for the same idea -->
  <ContainerDefaultArgs Include="database;update" />
</ItemGroup>
```

### `ContainerAppCommandInstruction`

The app command instruction configuration helps control the way the `ContainerEntrypoint`, `ContainerEntrypointArgs`, `ContainerAppCommand`, `ContainerAppCommandArgs`, and `ContainerDefaultArgs` are combined to form the final command that is run in the container. This depends greatly on if an `ENTRYPOINT` is present in the base image. This property takes one of three values: `"DefaultArgs"`, `"Entrypoint"`, or `"None"`.

- `Entrypoint`:
  - In this mode, the entrypoint is defined by `ContainerAppCommand`, `ContainerAppCommandArgs`, and `ContainerDefaultArgs`.
- `None`:
  - In this mode, the entrypoint is defined by `ContainerEntrypoint`, `ContainerEntrypointArgs`, and `ContainerDefaultArgs`.
- `DefaultArgs`:
  - This is the most complex mode—if none of the `ContainerEntrypoint[Args]` items are present, the `ContainerAppCommand[Args]` and `ContainerDefaultArgs` are used to create the entrypoint and command. The base image entrypoint for base images that have it hard-coded to `dotnet` or `/usr/bin/dotnet` is skipped so that you have complete control.
  - If both `ContainerEntrypoint` and `ContainerAppCommand` are present, then `ContainerEntrypoint` becomes the entrypoint, and `ContainerAppCommand` becomes the command.

> [!NOTE]
> The `ContainerEntrypoint` and `ContainerEntrypointArgs` configuration items have been deprecated as of .NET 8.

> [!IMPORTANT]
> This is for advanced users-most apps shouldn't need to customize their entrypoint to this degree. For more information and if you'd like to provide use cases for your scenarios, see [GitHub: .NET SDK container builds discussions](https://github.com/dotnet/sdk-container-builds/discussions).

### `ContainerUser`

The user configuration property controls the default user that the container runs as. This is often used to run the container as a nonroot user, which is a best practice for security. There are a few constraints for this configuration to be aware of:

- It can take various forms—username, linux user ids, group name, linux group id, `username:groupname`, and other ID variants.
- There's no verification that the user or group specified exists on the image.
- Changing the user can alter the behavior of the app, especially in regards to things like _File System_ permissions.

The default value of this field varies by project TFM and target operating system:

- If you're targeting .NET 8 or higher and using the Microsoft runtime images, then:
  - on Linux the rootless user `app` is used (though it's referenced by its user ID)
  - on Windows the rootless user `ContainerUser` is used
- Otherwise, no default `ContainerUser` is used

```xml
<PropertyGroup>
  <ContainerUser>my-existing-app-user</ContainerUser>
</PropertyGroup>
```

> [!TIP]
> The `APP_UID` environment variable is used to set user information in your container. This value can come from environment variables defined in your base image (like that Microsoft .NET images do), or you can set it yourself via the `ContainerEnvironmentVariable` syntax.

:::zone-end
:::zone pivot="dotnet-7-0"

However, you can control how your app is executed by using `ContainerEntrypoint` and `ContainerEntrypointArgs`.

### `ContainerEntrypoint`

The container entry point can be used to customize the `ENTRYPOINT` of the container, which is the executable that is called when the container is started. By default, for builds that create an app host, it's set as the `ContainerEntrypoint`. For builds that don't create an executable, the `dotnet path/to/app.dll` is used as the `ContainerEntrypoint`.

The `ContainerEntrypoint` node has a single attribute:

- `Include`: The command, option, or argument to use in the `ContainerEntrypoint` command.

For example, consider the following sample .NET project item group:

```xml
<ItemGroup Label="Entrypoint Assignment">
  <!-- This is how you would start the dotnet ef tool in your container -->
  <ContainerEntrypoint Include="dotnet" />
  <ContainerEntrypoint Include="ef" />

  <!-- This shorthand syntax means the same thing.
       Note the semicolon separating the tokens. -->
  <ContainerEntrypoint Include="dotnet;ef" />
</ItemGroup>
```

### `ContainerEntrypointArgs`

The container entry point args node controls the default arguments provided to the `ContainerEntrypoint`. This should be used when the `ContainerEntrypoint` is a program that the user might want to use on its own. By default, no `ContainerEntrypointArgs` are created on your behalf.

The `ContainerEntrypointArg` node has a single attribute:

- `Include`: The option or argument to apply to the `ContainerEntrypoint` command.

Consider the following example .NET project item group:

```xml
<ItemGroup>
  <!-- Assuming the ContainerEntrypoint defined above,
       this would be the way to update the database by
       default, but let the user run a different EF command. -->
  <ContainerEntrypointArgs Include="database" />
  <ContainerEntrypointArgs Include="update" />

  <!-- This is the shorthand syntax for the same idea -->
  <ContainerEntrypointArgs Include="database;update" />
</ItemGroup>
```

:::zone-end

### Default container labels

Labels are often used to provide consistent metadata on container images. This package provides some default labels to encourage better maintainability of the generated images.

- `org.opencontainers.image.created` is set to the ISO 8601 format of the current UTC `DateTime`.

For more information, see [Implement conventional labels on top of existing label infrastructure](https://github.com/dotnet/sdk-container-builds/issues/96).

## Clean up resources

In this article, you published a .NET worker as a container image. If you want, delete this resource. Use the `docker images` command to see a list of installed images.

```console
docker images
```

Consider the following example output:

```console
REPOSITORY            TAG       IMAGE ID       CREATED          SIZE
dotnet-worker-image   1.0.0     25aeb97a2e21   12 seconds ago   191MB
```

> [!TIP]
> Image files can be large. Typically, you would remove temporary containers you created while testing and developing your app. You usually keep the base images with the runtime installed if you plan on building other images based on that runtime.

To delete the image, copy the image ID and run the `docker image rm` command:

```console
docker image rm 25aeb97a2e21
```

## Next steps

- [Announcing built-in container support for the .NET SDK](https://devblogs.microsoft.com/dotnet/announcing-builtin-container-support-for-the-dotnet-sdk)
- [Tutorial: Containerize a .NET app](build-container.md)
- [.NET container images](container-images.md)
- [Review the Azure services that support containers](https://azure.microsoft.com/overview/containers/)
- [Read about Dockerfile commands](https://docs.docker.com/engine/reference/builder/)
- [Explore the container tools in Visual Studio](/visualstudio/containers/overview)
