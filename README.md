**Disclaimer**
I shamelessly copied the code and documentation from the original OctoPack project and changed it to conform to my way of building a package.

I only want to create a package, so no deployment from builds.  
Also, OctoPack already has the feature I needed, so this was a nice excercise for me to create a lean, no custom code, nuget to do the same thing.  I rely on Microsoft to do all the heavy lifting for me and by convention use tools that are readily available in visual studio.  Take a look at the targets file in the nuget to see how little there is.  I basically copied the original OctoPacks install and uninstall scripts, as it was sweetly simple as well.  All custom tasks have been removed.

**Upfront Requirements**
1. Your solution has to have nuget restore enabled.  Make sure there is a .nuget folder.  I assume that the Nuget.exe is in here.
2. You have to provide a `.nuspec`, more on that later.


**Pingo.OctoPack** is an open source project that makes it easy to create [Octopus Deploy](http://octopusdeploy.com)-compatible NuGet packages.
It is an alternative to OctopusDeploy/OctoPack, where I let Microsoft package the website first via a Web Deploy Package

Please refer to the following prior to going down this path;
[The Original OctoPack Project](https://github.com/OctopusDeploy/OctoPack "Packaging NuGet packages for Octopus")

In the end we both produce the exact same thing; an Octopus package.  How we get there differs.  I rely on Microsoft's Web Deploy Packaging to do the heavy lifting of packaging the website.  In my case, I have files that need to be packaged that do not exist in the project.  Using the original OctoPack doesn't work for me because it misses those files.  WebDeploy lets me configure my csproj using the following method;
[Include Extra Files for WebDeployment](http://sedodream.com/2010/05/01/WebDeploymentToolMSDeployBuildPackageIncludingExtraFilesOrExcludingSpecificFiles.aspx "")

Once that is done, we have a ready made folder ready to be turned into an Octopus package.

## Installing Pingo.OctoPack

Assuming you have an ASP.NET web site or Windows Service C# (or VB.NET) project, creating a NuGet package that works with Octopus is easy. 

1. Ensure you have installed NuGet into your Visual Studio
2. From the View menu, open Other Windows -> Package Manager Console
3. In the Default Project drop down, choose the ASP.NET web site or Windows Service project that you would like to package

Install the OctoPack package by typing:

    Install-Package Pingo.OctoPack 

You will see output similar to this:

![Installing OctoPack](https://s3.amazonaws.com/octopus-images/doc/octopack/octopack-install.png "Installing OctoPack")
 
## Building packages

## Adding a NuSpec

A `.nuspec` file describes the contents of your NuGet package. You have to have one.  I use a custom TextTransform(TT) to make sure my `.nuspec` always has the same version as my project assembly. Here is the spec; [simple .nuspec file](http://docs.nuget.org/docs/reference/nuspec-reference "NuSpec file format"). The file name should match the name of your C# project - for example, **Sample.Web.nuspec** if your ASP.NET project is named **Sample.Web**.

 Here is an example of the .nuspec file contents:

	<?xml version="1.0"?>
	<package xmlns="http://schemas.microsoft.com/packaging/2010/07/nuspec.xsd">
	  <metadata>
	    <id>Sample.Web</id>
	    <title>Your Web Application</title>
	    <version>1.0.6</version>
	    <authors>Your name</authors>
	    <owners>Your name</owners>
	    <licenseUrl>http://yourcompany.com</licenseUrl>
	    <projectUrl>http://yourcompany.com</projectUrl>
	    <requireLicenseAcceptance>false</requireLicenseAcceptance>
	    <description>A sample project</description>
	    <releaseNotes>This release contains the following changes...</releaseNotes>
	  </metadata>
	  <files>
      <file src="obj\Release\Package\PackageTmp\**\*.*" target="" />
    </files>
	</package>
	
Notice the `<files>` section.  
This is where the WebPublish puts the website.  This is our ready made, ready to package folder.

See the [NuSpec files reference documentation](http://docs.nuget.org/docs/reference/nuspec-reference#Specifying_Files_to_Include_in_the_Package) for more examples on how to specify which files to include.


## What is packaged?

Only a website and only when called from a very specific MSBUILD commandline.
When web applications are packaged, only the content from here are included;
*obj\Release\Package\PackageTmp\*

![ASP.NET package](https://s3.amazonaws.com/octopus-images/doc/octopack/octopack-new-package-web.png "ASP.NET package")

To have Pingo.OctoPack create a NuGet package from your build, set the `RunPingoOctoPack` MSBuild property to `true`. For example, if you are compiling from the command line, you might use:

    "c:\Program Files (x86)\MSBuild\12.0\Bin\MSBuild.exe" MySolution.sln /t:Build /p:DeployOnBuild=true;PublishProfile=WebDeployPackage /p:Configuration=Release /p:RunPingoOctoPack=true

After the build completes, in the $(TargetDir) you will find a NuGet package. This package is ready to be deployed using your [Octopus Deploy](http://octopusdeploy.com) server.

## Version numbers

NuGet packages have version numbers. 
When you use Pingo.OctoPack, the NuGet package version number will come from .nuspec file:  As I said earlier, I use TT to get it right.

## Publishing
My Opinion: DO NOT Publish directly from a build.  Publish only when you have completed your suite of unit tests and CI work.
In short, get your build pipeline in order.  I publish to a nuget feed, much later in my jobs.  The build is only a small part of what can be a very complicated pipeline.

