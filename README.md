# Web application that uses Open XML SDK to configure Office documents to automatically open a specified add-in

The are a number of scenarios in which an Office Add-in should be embedded in an Office document and automatically open when the document opens.

> **Note**: For more information about automatically opening an add-in in a document, see [Automatically open a task pane with a document](https://dev.office.com/docs/add-ins/develop/automatically-open-a-task-pane-with-a-document).

This sample uses .NET and the Open XML SDK to configure Office documents with the markup that makes this possible. In this sample, the [Script Lab add-in](https://store.office.com/en-001/app.aspx?assetid=WA104380862&sourcecorrid=d5eb16ba-d44c-41f5-892e-96d383be4393&searchapppos=0&ui=en-US&rs=en-001&ad=US&appredirect=false) is embedded in an Office file of the user's choice.


## Table of Contents
* [Change History](#change-history)
* [Prerequisites](#prerequisites)
* [To use the project](#to-use-the-project)
* [About the code](#about-the-code)
* [Questions and comments](#questions-and-comments)
* [Additional resources](#additional-resources)

## Change History

June 15, 2017:

* Initial version.

## Prerequisites

* Office 2016, Version 1705, build 16.0.8122.1000 Click-to-Run, or later.
* Visual Studio 2017
* [Open XML SDK 2.5](https://www.microsoft.com/en-us/download/details.aspx?id=30425)

## To use the project

1. Clone or download this repo.
2. Open the Office-OOXML-EmbedAddin.sln file in Visual Studio.
3. In Visual Studio add references to these assemblies:

    - WindowsBase (in the **Framework** tab of the Visual Studio Extensions Manager)
    - DocumentFormat.OpenXml (in the **Extensions** tab) 
4. Press F5. This opens the home page of the application in your browser.
5. Use the **Browse** control on the page to navigate to any Excel, Word, or PowerPoint file. *Do not use a file that is already configured to open an add-in automatically. Only one add-in can be auto-opened in a file.*
6. Press **Upload**.
7. Optional. If there is a particular snippet that you want to import as soon as Script Lab runs, enter its Gist ID in the textbox.
   > **Note**: This feature may not be supported in Script Lab for a short period after this sample is released. But it will do no harm to enter a Gist ID.

8. Press **Embed Script Lab**.
9. Press **Download** and follow your browser's prompts to open or save the file. When the file opens, the task pane opens and near the top, you are prompted to trust Script Lab (unless you already had it installed). When you do, Script Lab opens. 
10. To verify that Script Lab is embedded in the document, close the file and reopen it. Script Lab opens in the task pane immediately.

## About the code

The file **Home.aspx.cs**:
- Provides the button event handlers and basic UI manipulation.
- Uses standard ASP.NET techniques to upload and download the file.
- Uses the file name extension of the uploaded file (xlsx, docx, or pptx) to determine the type of file. This needs to be done at the outset because the Open XML SDK generally has distinct APIs for each type of file.
- Calls into the OOXMLHelper to validate the file and calls into the AddInEmbedder to embed Script Lab in the file and set to automatically open.

The file **AddInEmbedder.cs**:
- Provides the main business logic, which in this sample is a method that embeds Script Lab.
- It makes calls into the OOXML helper based on the type of the file.

The file **OOXMLHelper.cs**:
- Provides all the detailed OOXML manipulation.
- Uses a standard technique for validating the Office file, which is simply to call the *Document.Open method on it. If the file is invalid, the method throws an exception.
- Contains mainly code that was generated by the **Open XML 2.5 SDK Productivity Tools** which are available at the link for the Open XML 2.5 SDK above.

There are two critical lines in this file.

First, is the line in the `GenerateWebExtensionPart1Content` method that sets the reference to Script Lab (strictly speaking, to the manifest for Script Lab): 
```
We.WebExtensionStoreReference webExtensionStoreReference1 = new We.WebExtensionStoreReference() { Id = "wa104380862", Version = "1.1.0.0", Store = "en-US", StoreType = "OMEX" };
```
In this code: 
- The **StoreType** value is "OMEX", an alias for the Office Store. 
- The **Store** value is "en-US" the culture section of the store where Script Lab is.
- The **Id** value is the Office Store's asset ID for Script Lab.

If you were setting up an add-in from a file share catalog for auto-open, you would use different values:
- The **StoreType** value would be "FileSystem". 
- The **Store** value would be the URL of the network share; for example, "\\\MyComputer\MySharedFolder".
- The **Id** value would be the app ID in the add-ins manifest.

> **Note**: For more information about alternative values for these attributes, see [Automatically open a task pane with a document](https://dev.office.com/docs/add-ins/develop/automatically-open-a-task-pane-with-a-document).

Second, is the line in the `GeneratePartContent` method that specifies the visibility of the taskpane when the file opens. 

```
Wetp.WebExtensionTaskpane webExtensionTaskpane1 = new Wetp.WebExtensionTaskpane() { DockState = "right", Visibility = true, Width = 350D, Row = (UInt32Value)4U };
```

In this code, the `Visibility` property of the `WebExtensionTaskpane` object is set to `true`. The effect of this is that the very first time that the file is opened after the code is run, the task pane opens with Script Lab in it (after the user accepts the prompt to trust Script Lab). This is what we want for this sample. However, in most scenarios you will probably want this set to `false`. The effect of setting it to false is that the *first* time the file is opened, the user has to install the add-in, from the **Add-in** button on the ribbon. On every *subsequent* opening of the file, the taskpane with the add-in opens automatically. 

The advantage of setting this property to `false` is that you can use the Office.js to turn give users the ability to turn on and off the auto-opening of the add-in. Specifically, your script sets the **Office.AutoShowTaskpaneWithDocument** document setting to `true` or `false`. However, if `WebExtensionTaskpane.Visibility` is set to `true`, there is no way for Office.js or, hence, your users to turn off the auto-opening of the add-in. Only editing the OOXML of the document can change `WebExtensionTaskpane.Visibility` to false.

> **Note**: For more information about task pane visibility at the level of the Open XML that these .NET APIs represent, see [Automatically open a task pane with a document](https://dev.office.com/docs/add-ins/develop/automatically-open-a-task-pane-with-a-document).

## Questions and comments

We'd love to get your feedback about this sample. You can send your feedback to us in the *Issues* section of this repository.

Questions about Microsoft Office 365 development in general should be posted to [Stack Overflow](http://stackoverflow.com/questions/tagged/office-js). If your question is about the Office JavaScript APIs, make sure that your questions are tagged with [office-js].

## Additional resources

* [Office add-in documentation](https://msdn.microsoft.com/en-us/library/office/jj220060.aspx)
* [Office Dev Center](http://dev.office.com/)
* More Office Add-in samples at [OfficeDev on Github](https://github.com/officedev)
* [Script Lab](https://aka.ms/scriptlab)

## Copyright
Copyright (c) 2017 Microsoft Corporation. All rights reserved.

