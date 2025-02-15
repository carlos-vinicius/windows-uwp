---
description: This article explains how to support the Share contract in a Universal Windows Platform (UWP) app.
title: Share data
ms.assetid: 32287F5E-EB86-4B98-97FF-8F6228D06782
ms.date: 02/08/2017
ms.topic: article
keywords: windows 10, uwp
ms.localizationpriority: medium
---
# Share data


This article explains how to support the Share contract in a Universal Windows Platform (UWP) app. The Share contract is an easy way to quickly share data, such as text, links, photos, and videos, between apps. For example, a user might want to share a webpage with their friends using a social networking app, or save a link in a notes app to refer to later.

## Set up an event handler

Add a [**DataRequested**](https://docs.microsoft.com/uwp/api/windows.applicationmodel.datatransfer.datatransfermanager.datarequested) event handler to be called whenever a user invokes share. This can occur either when the user taps a control in your app (such as a button or app bar command) or automatically in a specific scenario (if the user finishes a level and gets a high score, for example).

[!code-cs[Main](./code/share_data/cs/MainPage.xaml.cs#SnippetPrepareToShare)]

When a [**DataRequested**](https://docs.microsoft.com/uwp/api/windows.applicationmodel.datatransfer.datatransfermanager.datarequested) event occurs, your app receives a [**DataRequest**](https://docs.microsoft.com/uwp/api/Windows.ApplicationModel.DataTransfer.DataRequest) object. This contains a [**DataPackage**](https://docs.microsoft.com/uwp/api/Windows.ApplicationModel.DataTransfer.DataPackage) that you can use to provide the content that the user wants to share. You must provide a title and data to share. A description is optional, but recommended.

[!code-cs[Main](./code/share_data/cs/MainPage.xaml.cs#SnippetCreateRequest)]

## Choose data

You can share various types of data, including:

-   Plain text
-   Uniform Resource Identifiers (URIs)
-   HTML
-   Formatted text
-   Bitmaps
-   Files
-   Custom developer-defined data

The [**DataPackage**](https://docs.microsoft.com/uwp/api/Windows.ApplicationModel.DataTransfer.DataPackage) object can contain one or more of these formats, in any combination. The following example demonstrates sharing text.

[!code-cs[Main](./code/share_data/cs/MainPage.xaml.cs#SnippetSetContent)]

## Set properties

When you package data for sharing, you can supply a variety of properties that provide additional information about the content being shared. These properties help target apps improve the user experience. For example, a description helps when the user is sharing content with more than one app. Adding a thumbnail when sharing an image or a link to a web page provides a visual reference to the user. For more information, see [**DataPackagePropertySet**](https://docs.microsoft.com/uwp/api/Windows.ApplicationModel.DataTransfer.DataPackagePropertySet).

All properties except the title are optional. The title property is mandatory and must be set.

[!code-cs[Main](./code/share_data/cs/MainPage.xaml.cs#SnippetSetProperties)]

## Launch the share UI

A UI for sharing is provided by the system. To launch it, call the [**ShowShareUI**](https://docs.microsoft.com/uwp/api/windows.applicationmodel.datatransfer.datatransfermanager.showshareui) method.

[!code-cs[Main](./code/share_data/cs/MainPage.xaml.cs#SnippetShowUI)]

## Handle errors

In most cases, sharing content is a straightforward process. However, there's always a chance that something unexpected could happen. For example, the app might require the user to select content for sharing but the user didn't select any. To handle these situations, use the [**FailWithDisplayText**](https://msdn.microsoft.com/library/windows/apps/Windows.ApplicationModel.DataTransfer.DataRequest.FailWithDisplayText(System.String)) method, which will display a message to the user if something goes wrong.

## Delay share with delegates

Sometimes, it might not make sense to prepare the data that the user wants to share right away. For example, if your app supports sending a large image file in several different possible formats, it's inefficient to create all those images before the user makes their selection.

To solve this problem, a [**DataPackage**](https://docs.microsoft.com/uwp/api/Windows.ApplicationModel.DataTransfer.DataPackage) can contain a delegate — a function that is called when the receiving app requests data. We recommend using a delegate any time that the data a user wants to share is resource-intensive.

<!-- For some reason, this snippet was inline in the WDCML topic. Suggest moving to VS project with rest of snippets. -->
```cs
async void OnDeferredImageRequestedHandler(DataProviderRequest request)
{
    // Provide updated bitmap data using delayed rendering
    if (this.imageStream != null)
    {
        DataProviderDeferral deferral = request.GetDeferral();
        InMemoryRandomAccessStream inMemoryStream = new InMemoryRandomAccessStream();

        // Decode the image.
        BitmapDecoder imageDecoder = await BitmapDecoder.CreateAsync(this.imageStream);

        // Re-encode the image at 50% width and height.
        BitmapEncoder imageEncoder = await BitmapEncoder.CreateForTranscodingAsync(inMemoryStream, imageDecoder);
        imageEncoder.BitmapTransform.ScaledWidth = (uint)(imageDecoder.OrientedPixelWidth * 0.5);
        imageEncoder.BitmapTransform.ScaledHeight = (uint)(imageDecoder.OrientedPixelHeight * 0.5);
        await imageEncoder.FlushAsync();

        request.SetData(RandomAccessStreamReference.CreateFromStream(inMemoryStream));
        deferral.Complete();
    }
}
```

## See also 

* [App-to-app communication](index.md)
* [Receive data](receive-data.md)
* [DataPackage](https://docs.microsoft.com/uwp/api/windows.applicationmodel.datatransfer.datapackage)
* [DataPackagePropertySet](https://docs.microsoft.com/uwp/api/windows.applicationmodel.datatransfer.datapackagepropertyset)
* [DataRequest](https://docs.microsoft.com/uwp/api/windows.applicationmodel.datatransfer.datarequest)
* [DataRequested](https://docs.microsoft.com/uwp/api/windows.applicationmodel.datatransfer.datatransfermanager.datarequested)
* [FailWithDisplayText](https://docs.microsoft.com/uwp/api/windows.applicationmodel.datatransfer.datarequest.failwithdisplaytext)
* [ShowShareUi](https://docs.microsoft.com/uwp/api/windows.applicationmodel.datatransfer.datatransfermanager.showshareui)
 

