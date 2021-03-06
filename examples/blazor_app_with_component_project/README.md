# Blazor App with Component Project

This example handles the use case for a blazor app with some components in a different project. The custom colors for deployment are defined in the BlazorApp.

## Relevant files

All the following files have been modified slightly from the plain Blazor sample project. All changes are detailed below.

### ComponentWithStyle/ComponentWithStyle.csproj

```xml
<Project Sdk="Microsoft.NET.Sdk.Razor">

  <PropertyGroup>
    <TargetFramework>netstandard2.0</TargetFramework>
    <RazorLangVersion>3.0</RazorLangVersion>
    <LazyStyleSheets_ComponentStyleFolder>css/component_with_style</LazyStyleSheets_ComponentStyleFolder>
    <LazyStyleSheets_UseWebCompiler>false</LazyStyleSheets_UseWebCompiler>
    
  </PropertyGroup>
  <ItemGroup>
    <PackageReference Include="Excubo.Blazor.LazyStyleSheet" Version="2.0.5" />
    <PackageReference Include="Microsoft.AspNetCore.Components" Version="3.1.3" />
    <PackageReference Include="Microsoft.AspNetCore.Components.Web" Version="3.1.3" />
    <PackageReference Include="Microsoft.Build.Framework" Version="16.5.0" />
    <PackageReference Include="Microsoft.Build.Utilities.Core" Version="16.5.0" />
  </ItemGroup>

</Project>
```

- added `<PackageReference Include="Excubo.Blazor.LazyStyleSheet" Version="2.0.5" />`.
- `LazyStyleSheets` is configured to use a non-default output path `css/component_with_style`.
- `LazyStyleSheets` is configured to not use `webcompiler` (because we want to define our own colors in the BlazorApp).


### ComponentWithStyle/ComponentWithUserDefinedColor.razor

```html
<div class="component-style">
    This Blazor component is defined in the <strong>ComponentWithStyle</strong> package.
</div>
```

- we use a custom style called `component-style`.


### ComponentWithStyle/ComponentWithUserDefinedColor.razor.scss

```css
@import '../../default_values.scss';
.component-style {
    color: $variable
}
```

- we reference the file `../../default_values.scss` which contains the definition for the custom color stored in `variable`.
- we define the style used in `ComponentWithUserDefinedColor.razor`
- we use a variable instead of a hardcoded value

### BlazorApp/Startup.cs

```cs
using Excubo.Blazor.LazyStyleSheet;
// other imports omitted here

namespace BlazorApp
{
    public class Startup
    {
        public Startup(IConfiguration configuration)
        {
            Configuration = configuration;
        }

        public IConfiguration Configuration { get; }

        // This method gets called by the runtime. Use this method to add services to the container.
        // For more information on how to configure your application, visit https://go.microsoft.com/fwlink/?LinkID=398940
        public void ConfigureServices(IServiceCollection services)
        {
            services.AddRazorPages();
            services.AddServerSideBlazor();
            services.AddStyleSheetLazyLoading();
            services.AddSingleton<WeatherForecastService>();
        }

        // Omitted here, because no change: public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
    }
}
```

- added service `services.AddStyleSheetLazyLoading();` (uses namespace `Excubo.Blazor.LazyStyleSheet`)

### BlazorApp/App.razor

```html

<Router AppAssembly="@typeof(Program).Assembly">
    <Found Context="routeData">
        <RouteView RouteData="@routeData" DefaultLayout="@typeof(MainLayout)" />
    </Found>
    <NotFound>
        <LayoutView Layout="@typeof(MainLayout)">
            <p>Sorry, there's nothing at this address.</p>
        </LayoutView>
    </NotFound>
</Router>

<Excubo.Blazor.LazyStyleSheet.StyleSheets />
```

-  added the `<Excubo.Blazor.LazyStyleSheet.StyleSheets />` component.

### BlazorApp/_Imports.razor

```cs
@using System.Net.Http
@using Microsoft.AspNetCore.Authorization
@using Microsoft.AspNetCore.Components.Authorization
@using Microsoft.AspNetCore.Components.Forms
@using Microsoft.AspNetCore.Components.Routing
@using Microsoft.AspNetCore.Components.Web
@using Microsoft.JSInterop
@using BlazorApp
@using BlazorApp.Shared
@using ComponentWithStyle 
```

- added `@using ComponentWithStyle`

### BlazorApp/Pages/Index.razor

```html
@page "/"

<h1>Hello, world!</h1>

Welcome to your new app.

<ComponentWithUserDefinedColor />

<SurveyPrompt Title="How is Blazor working for you?" />
```

- use the `<ComponentWithUserDefinedColor />` component in the index page (so that we see it on the first page load).


### BlazorApp/wwwroot/default_values.scss

```css
$variable:blue;
```

- define our custom color `variable` to be `blue`.

### BlazorApp/BlazorApp.csproj

```xml
<Project Sdk="Microsoft.NET.Sdk.Web">

  <PropertyGroup>
    <TargetFramework>netcoreapp3.1</TargetFramework>
  </PropertyGroup>

  <ItemGroup>
    <ProjectReference Include="..\ComponentWithStyle\ComponentWithStyle.csproj" />
  </ItemGroup>

  <ItemGroup>
    <MaterialResources Include="$(MSBuildThisFileDirectory)..\ComponentWithStyle\wwwroot\css\component_with_style\*" />
  </ItemGroup>
  <Target Name="CopyResources" BeforeTargets="BeforeCompile">
    <Copy SourceFiles="@(MaterialResources)" DestinationFolder="$(MsBuildProjectDirectory)\wwwroot\css\component_with_style\" />
  </Target>

  <Target Name="CompileWithUserDefinedColors" AfterTargets="CoreCompile">
    <Exec Command="webcompiler -r wwwroot/css" />
  </Target>

</Project>
```

- the BlazorApp has a dependency on project `ComponentWithStyle`: `<ProjectReference Include="..\ComponentWithStyle\ComponentWithStyle.csproj" />`
- Since this is a component project, not a component nuget package, we have to copy the files ourselves. That's achieved by the `Target` CopyResources:
```xml
  <Target Name="CopyResources" BeforeTargets="BeforeCompile">
    <Copy SourceFiles="@(MaterialResources)" DestinationFolder="$(MsBuildProjectDirectory)\wwwroot\css\component_with_style\" />
  </Target>
```
 - Now that we have the style sheets from the component project and have the variables defined (`BlazorApp/wwwroot/default_values.scss`), we want `webcompiler` to compile those scss files:
 ```xml
  <Target Name="CompileWithUserDefinedColors" AfterTargets="CoreCompile">
    <Exec Command="webcompiler -r wwwroot/css" />
  </Target>
 ```


