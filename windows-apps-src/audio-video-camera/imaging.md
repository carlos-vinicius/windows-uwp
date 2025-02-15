---
ms.assetid: 3FD2AA71-EF67-47B2-9332-3FFA5D3703EA
description: This article explains how to load and save image files using BitmapDecoder and BitmapEncoder and how to use the SoftwareBitmap object to represent bitmap images.
title: Create, edit, and save bitmap images
ms.date: 03/22/2018
ms.topic: article
keywords: windows 10, uwp
ms.localizationpriority: medium
---
# Create, edit, and save bitmap images



This article explains how to load and save image files using [**BitmapDecoder**](https://docs.microsoft.com/uwp/api/Windows.Graphics.Imaging.BitmapDecoder) and [**BitmapEncoder**](https://docs.microsoft.com/uwp/api/Windows.Graphics.Imaging.BitmapEncoder) and how to use the [**SoftwareBitmap**](https://docs.microsoft.com/uwp/api/Windows.Graphics.Imaging.SoftwareBitmap) object to represent bitmap images.

The **SoftwareBitmap** class is a versatile API that can be created from multiple sources including image files, [**WriteableBitmap**](https://docs.microsoft.com/uwp/api/Windows.UI.Xaml.Media.Imaging.WriteableBitmap) objects, Direct3D surfaces, and code. **SoftwareBitmap** allows you to easily convert between different pixel formats and alpha modes, and allows low-level access to pixel data. Also, **SoftwareBitmap** is a common interface used by multiple features of Windows, including:

-   [**CapturedFrame**](https://docs.microsoft.com/uwp/api/Windows.Media.Capture.CapturedFrame) allows you to get frames captured by the camera as a **SoftwareBitmap**.

-   [**VideoFrame**](https://docs.microsoft.com/uwp/api/Windows.Media.VideoFrame) allows you to get a **SoftwareBitmap** representation of a **VideoFrame**.

-   [**FaceDetector**](https://docs.microsoft.com/uwp/api/Windows.Media.FaceAnalysis.FaceDetector) allows you to detect faces in a **SoftwareBitmap**.

The sample code in this article uses APIs from the following namespaces.

[!code-cs[Namespaces](./code/ImagingWin10/cs/MainPage.xaml.cs#SnippetNamespaces)]

## Create a SoftwareBitmap from an image file with BitmapDecoder

To create a [**SoftwareBitmap**](https://docs.microsoft.com/uwp/api/Windows.Graphics.Imaging.SoftwareBitmap) from a file, get an instance of [**StorageFile**](https://docs.microsoft.com/uwp/api/Windows.Storage.StorageFile) containing the image data. This example uses a [**FileOpenPicker**](https://docs.microsoft.com/uwp/api/Windows.Storage.Pickers.FileOpenPicker) to allow the user to select an image file.

[!code-cs[PickInputFile](./code/ImagingWin10/cs/MainPage.xaml.cs#SnippetPickInputFile)]

Call the [**OpenAsync**](https://docs.microsoft.com/uwp/api/windows.storage.istoragefile.openasync) method of the **StorageFile** object to get a random access stream containing the image data. Call the static method [**BitmapDecoder.CreateAsync**](https://docs.microsoft.com/uwp/api/windows.graphics.imaging.bitmapdecoder.createasync) to get an instance of the [**BitmapDecoder**](https://docs.microsoft.com/uwp/api/Windows.Graphics.Imaging.BitmapDecoder) class for the specified stream. Call [**GetSoftwareBitmapAsync**](https://docs.microsoft.com/uwp/api/windows.graphics.imaging.bitmapdecoder.getsoftwarebitmapasync) to get a [**SoftwareBitmap**](https://docs.microsoft.com/uwp/api/Windows.Graphics.Imaging.SoftwareBitmap) object containing the image.

[!code-cs[CreateSoftwareBitmapFromFile](./code/ImagingWin10/cs/MainPage.xaml.cs#SnippetCreateSoftwareBitmapFromFile)]

## Save a SoftwareBitmap to a file with BitmapEncoder

To save a **SoftwareBitmap** to a file, get an instance of **StorageFile** to which the image will be saved. This example uses a [**FileSavePicker**](https://docs.microsoft.com/uwp/api/Windows.Storage.Pickers.FileSavePicker) to allow the user to select an output file.

[!code-cs[PickOutputFile](./code/ImagingWin10/cs/MainPage.xaml.cs#SnippetPickOutputFile)]

Call the [**OpenAsync**](https://docs.microsoft.com/uwp/api/windows.storage.istoragefile.openasync) method of the **StorageFile** object to get a random access stream to which the image will be written. Call the static method [**BitmapEncoder.CreateAsync**](https://docs.microsoft.com/uwp/api/windows.graphics.imaging.bitmapencoder.createasync) to get an instance of the [**BitmapEncoder**](https://docs.microsoft.com/uwp/api/Windows.Graphics.Imaging.BitmapEncoder) class for the specified stream. The first parameter to **CreateAsync** is a GUID representing the codec that should be used to encode the image. **BitmapEncoder** class exposes a property containing the ID for each codec supported by the encoder, such as [**JpegEncoderId**](https://docs.microsoft.com/uwp/api/windows.graphics.imaging.bitmapencoder.jpegencoderid).

Use the [**SetSoftwareBitmap**](https://docs.microsoft.com/uwp/api/windows.graphics.imaging.bitmapencoder.setsoftwarebitmap) method to set the image that will be encoded. You can set values of the [**BitmapTransform**](https://docs.microsoft.com/uwp/api/Windows.Graphics.Imaging.BitmapTransform) property to apply basic transforms to the image while it is being encoded. The [**IsThumbnailGenerated**](https://docs.microsoft.com/uwp/api/windows.graphics.imaging.bitmapencoder.isthumbnailgenerated) property determines whether a thumbnail is generated by the encoder. Note that not all file formats support thumbnails, so if you use this feature, you should catch the unsupported operation error that will be thrown if thumbnails are not supported.

Call [**FlushAsync**](https://docs.microsoft.com/uwp/api/windows.graphics.imaging.bitmapencoder.flushasync) to cause the encoder to write the image data to the specified file.

[!code-cs[SaveSoftwareBitmapToFile](./code/ImagingWin10/cs/MainPage.xaml.cs#SnippetSaveSoftwareBitmapToFile)]

You can specify additional encoding options when you create the **BitmapEncoder** by creating a new [**BitmapPropertySet**](https://docs.microsoft.com/uwp/api/Windows.Graphics.Imaging.BitmapPropertySet) object and populating it with one or more [**BitmapTypedValue**](https://docs.microsoft.com/uwp/api/Windows.Graphics.Imaging.BitmapTypedValue) objects representing the encoder settings. For a list of supported encoder options, see [BitmapEncoder options reference](bitmapencoder-options-reference.md).

[!code-cs[UseEncodingOptions](./code/ImagingWin10/cs/MainPage.xaml.cs#SnippetUseEncodingOptions)]

## Use SoftwareBitmap with a XAML Image control

To display an image within a XAML page using the [**Image**](https://docs.microsoft.com/uwp/api/Windows.UI.Xaml.Controls.Image) control, first define an **Image** control in your XAML page.

[!code-xml[ImageControl](./code/ImagingWin10/cs/MainPage.xaml#SnippetImageControl)]

Currently, the **Image** control only supports images that use BGRA8 encoding and pre-multiplied or no alpha channel. Before attempting to display an image, test to make sure it has the correct format, and if not, use the **SoftwareBitmap** static [**Convert**](https://docs.microsoft.com/uwp/api/windows.graphics.imaging.softwarebitmap.windows) method to convert the image to the supported format.

Create a new [**SoftwareBitmapSource**](https://docs.microsoft.com/uwp/api/Windows.UI.Xaml.Media.Imaging.SoftwareBitmapSource) object. Set the contents of the source object by calling [**SetBitmapAsync**](https://docs.microsoft.com/uwp/api/windows.ui.xaml.media.imaging.softwarebitmapsource.setbitmapasync), passing in a **SoftwareBitmap**. Then you can set the [**Source**](https://docs.microsoft.com/uwp/api/windows.ui.xaml.controls.image.source) property of the **Image** control to the newly created **SoftwareBitmapSource**.

[!code-cs[SoftwareBitmapToWriteableBitmap](./code/ImagingWin10/cs/MainPage.xaml.cs#SnippetSoftwareBitmapToWriteableBitmap)]

You can also use **SoftwareBitmapSource** to set a **SoftwareBitmap** as the [**ImageSource**](https://docs.microsoft.com/uwp/api/windows.ui.xaml.media.imagebrush.imagesource) for an [**ImageBrush**](https://docs.microsoft.com/uwp/api/Windows.UI.Xaml.Media.ImageBrush).

## Create a SoftwareBitmap from a WriteableBitmap

You can create a **SoftwareBitmap** from an existing **WriteableBitmap** by calling [**SoftwareBitmap.CreateCopyFromBuffer**](https://docs.microsoft.com/uwp/api/windows.graphics.imaging.softwarebitmap.createcopyfrombuffer) and supplying the **PixelBuffer** property of the **WriteableBitmap** to set the pixel data. The second argument allows you to request a pixel format for the newly created **WriteableBitmap**. You can use the [**PixelWidth**](https://docs.microsoft.com/uwp/api/windows.ui.xaml.media.imaging.bitmapsource.pixelwidth) and [**PixelHeight**](https://docs.microsoft.com/uwp/api/windows.ui.xaml.media.imaging.bitmapsource.pixelheight) properties of the **WriteableBitmap** to specify the dimensions of the new image.

[!code-cs[WriteableBitmapToSoftwareBitmap](./code/ImagingWin10/cs/MainPage.xaml.cs#SnippetWriteableBitmapToSoftwareBitmap)]

## Create or edit a SoftwareBitmap programmatically

So far this topic has addressed working with image files. You can also create a new **SoftwareBitmap** programatically in code and use the same technique to access and modify the **SoftwareBitmap**'s pixel data.

**SoftwareBitmap** uses COM interop to expose the raw buffer containing the pixel data.

To use COM interop, you must include a reference to the **System.Runtime.InteropServices** namespace in your project.

[!code-cs[InteropNamespace](./code/ImagingWin10/cs/MainPage.xaml.cs#SnippetInteropNamespace)]

Initialize the [**IMemoryBufferByteAccess**](https://docs.microsoft.com/previous-versions//mt297505(v=vs.85)) COM interface by adding the following code within your namespace.

[!code-cs[COMImport](./code/ImagingWin10/cs/MainPage.xaml.cs#SnippetCOMImport)]

Create a new **SoftwareBitmap** with pixel format and size you want. Or, use an existing **SoftwareBitmap** for which you want to edit the pixel data. Call [**SoftwareBitmap.LockBuffer**](https://docs.microsoft.com/uwp/api/windows.graphics.imaging.softwarebitmap.lockbuffer) to obtain an instance of the [**BitmapBuffer**](https://docs.microsoft.com/uwp/api/Windows.Graphics.Imaging.BitmapBuffer) class representing the pixel data buffer. Cast the **BitmapBuffer** to the **IMemoryBufferByteAccess** COM interface and then call [**IMemoryBufferByteAccess.GetBuffer**](https://docs.microsoft.com/windows/desktop/WinRT/imemorybufferbyteaccess-getbuffer) to populate a byte array with data. Use the [**BitmapBuffer.GetPlaneDescription**](https://docs.microsoft.com/uwp/api/windows.graphics.imaging.bitmapbuffer.getplanedescription) method to get a [**BitmapPlaneDescription**](https://docs.microsoft.com/uwp/api/Windows.Graphics.Imaging.BitmapPlaneDescription) object that will help you calculate the offset into the buffer for each pixel.

[!code-cs[CreateNewSoftwareBitmap](./code/ImagingWin10/cs/MainPage.xaml.cs#SnippetCreateNewSoftwareBitmap)]

Because this method accesses the raw buffer underlying the Windows Runtime types, it must be declared using the **unsafe** keyword. You must also configure your project in Microsoft Visual Studio to allow the compilation of unsafe code by opening the project's **Properties** page, clicking the **Build** property page, and selecting the **Allow Unsafe Code** checkbox.

## Create a SoftwareBitmap from a Direct3D surface

To create a **SoftwareBitmap** object from a Direct3D surface, you must include the [**Windows.Graphics.DirectX.Direct3D11**](https://docs.microsoft.com/uwp/api/Windows.Graphics.DirectX.Direct3D11) namespace in your project.

[!code-cs[Direct3DNamespace](./code/ImagingWin10/cs/MainPage.xaml.cs#SnippetDirect3DNamespace)]

Call [**CreateCopyFromSurfaceAsync**](https://docs.microsoft.com/uwp/api/windows.graphics.imaging.softwarebitmap.createcopyfromsurfaceasync) to create a new **SoftwareBitmap** from the surface. As the name indicates, the new **SoftwareBitmap** has a separate copy of the image data. Modifications to the **SoftwareBitmap** will not have any effect on the Direct3D surface.

[!code-cs[CreateSoftwareBitmapFromSurface](./code/ImagingWin10/cs/MainPage.xaml.cs#SnippetCreateSoftwareBitmapFromSurface)]

## Convert a SoftwareBitmap to a different pixel format

The **SoftwareBitmap** class provides the static method, [**Convert**](https://docs.microsoft.com/uwp/api/windows.graphics.imaging.softwarebitmap.windows), that allows you to easily create a new **SoftwareBitmap** that uses the pixel format and alpha mode you specify from an existing **SoftwareBitmap**. Note that the newly created bitmap has a separate copy of the image data. Modifications to the new bitmap will not affect the source bitmap.

[!code-cs[Convert](./code/ImagingWin10/cs/MainPage.xaml.cs#SnippetConvert)]

## Transcode an image file

You can transcode an image file directly from a [**BitmapDecoder**](https://docs.microsoft.com/uwp/api/Windows.Graphics.Imaging.BitmapDecoder) to a [**BitmapEncoder**](https://docs.microsoft.com/uwp/api/Windows.Graphics.Imaging.BitmapEncoder). Create a [**IRandomAccessStream**](https://docs.microsoft.com/uwp/api/Windows.Storage.Streams.IRandomAccessStream) from the file to be transcoded. Create a new **BitmapDecoder** from the input stream. Create a new [**InMemoryRandomAccessStream**](https://docs.microsoft.com/uwp/api/Windows.Storage.Streams.InMemoryRandomAccessStream) for the encoder to write to and call [**BitmapEncoder.CreateForTranscodingAsync**](https://docs.microsoft.com/uwp/api/windows.graphics.imaging.bitmapencoder.createfortranscodingasync), passing in the in-memory stream and the decoder object. Encode options are not supported when transcoding; instead you should use [**CreateAsync**](https://docs.microsoft.com/uwp/api/windows.graphics.imaging.bitmapencoder.createasync). Any properties in the input image file that you do not specifically set on the encoder, will be written to the output file unchanged. Call [**FlushAsync**](https://docs.microsoft.com/uwp/api/windows.graphics.imaging.bitmapencoder.flushasync) to cause the encoder to encode to the in-memory stream. Finally, seek the file stream and the in-memory stream to the beginning and call [**CopyAsync**](https://docs.microsoft.com/uwp/api/windows.storage.streams.randomaccessstream.copyasync) to write the in-memory stream out to the file stream.

[!code-cs[TranscodeImageFile](./code/ImagingWin10/cs/MainPage.xaml.cs#SnippetTranscodeImageFile)]

## Related topics

* [BitmapEncoder options reference](bitmapencoder-options-reference.md)
* [Image Metadata](image-metadata.md)
 

 




