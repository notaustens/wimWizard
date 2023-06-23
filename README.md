# Step 1: Environment Preparation

In order to create custom operating system images, you must first set up a folder structure on your system to facilitate the process. This can be accomplished by following the steps outlined below:

1. Open up the command prompt (or PowerShell) as administrator.
2. Navigate to the root of your `C:\` drive by entering the following command

```
cd c:\
```

3. Create two folders; the first of which will act as a repository for the operating system install images (e.g., the `install.wim` file located in the sources folder of the target operating systems image file) and the second will act as the mount point for the images. This can be accomplished by entering the following commands in your terminal:

```
mkdir .\iso
mkdir .\mount
```

# Step 2: Preliminary Investigation

Now that we have a folder structure in place, we can begin the customization process by conducting an initial investigation of the Windows image file (`install.wim`).

1. Copy the `install.wim` to the `.\iso` folder you created in the previous step. This can be accomplished manually via the File Explorer or through the command prompt by mounting (or extracting the contents of) the target operating systems image file (`.iso`), opening up a command prompt in that location, and entering in the following command:

```
copy .\exampleInstall.wim c:\iso
```

2. Once you have copied the `install.wim` image file to the `c:\iso` directory, enter the following command in your command prompt to extract pertinent information (e.g., version index number, etc.) about the operating system image:

```
DISM /Get-ImageInfo /imagefile:"C:\iso\exampleInstall.wim"
```

# Step 3: Mount the Windows Image File

If the previous command executed successfully, you should see a list of the various versions of Windows embedded in the `install.wim` image file. Each version will be denoted by an index number that can be used to mount that specific edition. 

1. Once you have identified the index number of the edition of Windows you need, enter the following command in your command prompt: **[source](https://learn.microsoft.com/en-us/windows-hardware/manufacture/desktop/mount-and-modify-a-windows-image-using-dism?source=recommendations&view=windows-11#apply-an-image)** 

```
DISM /Mount-image /imagefile:"C:\iso\exampleInstall.wim" /Index:<targetEditionsIndexNumber> /MountDir:"C:\mount" /optimize
```

# Step 4: Investigate the Windows Image File

DISM is extremely powerful and can be used to prune unnecessary or superseded core components from the operating system image. Specifically, you can add or remove drivers, applications, packages, updates, language packs, features, and more. This is advantageous for a number of reasons, but in particular it reduces the size of the operating systems image file in addition to reducing the attack surface of the operating system image itself. 

1. To begin with, enter the following string in the command prompt to create a list of the Application Packages that have been included in the edition of Windows that you are working on: 

```
DISM /Image:"C:\mount" /Get-ProvisionedAppxPackages /format:table > "C:\iso\appx.txt" 
```

2. Next, create a list of the operating systems inherent capabilities by entering the following string in the command prompt:

```
DISM /Image:"C:\mount" /Get-Capabilities /format:table > "C:\iso\capabilities.txt"
```

3. As with the previous step, enter the following string in the command prompt to generate a list of the features that have been included in the operating system image: 

```
DISM /Image:"C:\mount" /Get-Features /format:table > "c:\iso\features.txt"
```

4. Last but not least, enter the following string into the command prompt to export a list of the packages that have been included in the operating system image:

```
DISM /Image:"C:\mount" /Get-Packages /format:table > "c:\iso\packages.txt"
```

# Step 5: Modify the Windows Image File

Once the previous steps have been completed, you should see four new text files located in the "c:\iso" folder. These text files can be used to identify undesirable applications, features, packages, and capabilities that have been embedded in the edition of windows that you are working on.

From here it becomes a bit tedious (and could surely be automated or optimized), but you basically need to just open up each text file and use the following commands to remove unnecessary components from the operating system image file.

## Remove Applications

1. Open the appx.txt file that was created in [Step 4](#step-4-investigate-the-windows-image-file) and use the "PackageName", in conjunction with the following command, to remove the application from the operating system image file:

```
DISM /Image:"C:\mount" /Remove-ProvisionedAppxPackage /PackageName:<PackageName>
```

2. Once the command executes successfully, repeat the process until all undesirable applications have been removed from the operating system image file.

## Remove Capabilities

1. Open the capabilities.txt file that was created in [Step 4](#step-4-investigate-the-windows-image-file) and use the "Capability Identity", in conjunction with the following command, to remove the capability from the Windows image file:

```
DISM /Image:"C:\mount" /Remove-Capability /CapabilityName:<Capability Identity>
```
2. Once the command executes successfully, repeat the process until all unnecessary capabilities have been removed from the Windows image file.

## Disable Features

1. Open the features.txt file that was created in [Step 4](#step-4-investigate-the-windows-image-file) and use the "Feature Name", in conjunction with the following command, to disable the feature present in the Windows image file:

```
DISM /Image:"C:\mount" /Disable-Feature /FeatureName:<Feature Name>
```
2. Once the command executes successfully, repeat the process until all unnecessary features have been disabled on the Windows image file.

## Remove Packages

1. Open the packages.txt file that was created in [Step 4](#step-4-investigate-the-windows-image-file) and use the "Package Identity" of the package to be removed, in conjunction with the following command, to remove the package from the Windows image file:

```
DISM /Image:"C:\mount" /Remove-Package /PackageName:<Package Identity>
```

2. Once the command executes successfully, repeat the process until all undesirable packages have been removed on the Windows image file.

# Step 6: Optimize the Windows Image File & Commit Changes

At this point in the process, all that remains is to use DISM to reduce the footprint of the Windows image file by cleaning up superseded components, resetting the base of the superseeded components, and then exporting the image to a new image file.

1. In the command prompt, enter the following command to cleanup the component base and optimize the Windows image file:

```
DISM /Image:"C:\mount\" /Cleanup-Image /StartComponentCleanup /ResetBase
```

2. Next, enter the following string in the command prompt to commit all of the changes that were made in the previous steps and unmount the Windows image file:

```
DISM /Unmount-Image /MountDir:"C:\mount\" /Commit
```

3. Upon successful execution of the previous command (e.g., the Windows image file was successfully unmounted), enter the following string in the command prompt to export the modified edition of Windows to its own Windows image file:

```
DISM /Export-Image /SourceImageFile:"C:\iso\exampleInstall.wim" /Index:<targetEditionsIndexNumber> /DestinationImageFile:"C:\iso\modifiedInstall.wim"
```
