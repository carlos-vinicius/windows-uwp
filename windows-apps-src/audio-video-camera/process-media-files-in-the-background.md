---
ms.assetid: B5E3A66D-0453-4D95-A3DB-8E650540A300
description: This article shows you how to use the MediaProcessingTrigger and a background task to process media files in the background.
title: Process media files in the background
ms.date: 02/08/2017
ms.topic: article
keywords: windows 10, uwp
ms.localizationpriority: medium
---
# Process media files in the background



This article shows you how to use the [**MediaProcessingTrigger**](https://docs.microsoft.com/uwp/api/Windows.ApplicationModel.Background.MediaProcessingTrigger) and a background task to process media files in the background.

The example app described in this article allows the user to select an input media file to transcode and specify an output file for the transcoding result. Then, a background task is launched to perform the transcoding operation. The [**MediaProcessingTrigger**](https://docs.microsoft.com/uwp/api/Windows.ApplicationModel.Background.MediaProcessingTrigger) is intended to support many different media processing scenarios besides transcoding, including rendering media compositions to disk and uploading processed media files after processing is complete.

For more detailed information on the different Universal Windows app features utilized in this sample, see:

-   [Transcode media files](transcode-media-files.md)
-   [Launching resuming and background tasks](https://docs.microsoft.com/windows/uwp/launch-resume/index)
-   [Tiles badges and notifications](https://docs.microsoft.com/windows/uwp/controls-and-patterns/tiles-badges-notifications)

## Create a media processing background task

To add a background task to your existing solution in Microsoft Visual Studio, Enter a name for your comp

1.  From the **File** menu, select **Add** and then **New Project...**.
2.  Select the project type **Windows Runtime Component (Universal Windows)**.
3.  Enter a name for your new component project. This example uses the project name **MediaProcessingBackgroundTask**.
4.  Click OK.

In **Solution Explorer**, right-click the icon for the "Class1.cs" file that is created by default and select **Rename**. Rename the file to "MediaProcessingTask.cs". When Visual Studio asks if you want to rename all of the references to this class, click **Yes**.

In the renamed class file, add the following **using** directives to include these namespaces in your project.
                                  
[!code-cs[BackgroundUsing](./code/MediaProcessingTriggerWin10/cs/MediaProcessingBackgroundTask/MediaProcessingTask.cs#SnippetBackgroundUsing)]

Update your class declaration to make your class inherit from [**IBackgroundTask**](https://docs.microsoft.com/uwp/api/Windows.ApplicationModel.Background.IBackgroundTask).

[!code-cs[BackgroundClass](./code/MediaProcessingTriggerWin10/cs/MediaProcessingBackgroundTask/MediaProcessingTask.cs#SnippetBackgroundClass)]

Add the following member variables to your class:

-   An [**IBackgroundTaskInstance**](https://docs.microsoft.com/uwp/api/Windows.ApplicationModel.Background.IBackgroundTaskInstance) that will be used to update the foreground app with the progress of the background task.
-   A [**BackgroundTaskDeferral**](https://docs.microsoft.com/uwp/api/Windows.ApplicationModel.Background.BackgroundTaskDeferral) that keeps the system from shutting down your background task while media transcoding is being performed asynchronously.
-   A **CancellationTokenSource** object that can be used to cancel the asynchronous transcoding operation.
-   The [**MediaTranscoder**](https://docs.microsoft.com/uwp/api/Windows.Media.Transcoding.MediaTranscoder) object that will be used to transcode media files.

[!code-cs[BackgroundMembers](./code/MediaProcessingTriggerWin10/cs/MediaProcessingBackgroundTask/MediaProcessingTask.cs#SnippetBackgroundMembers)]

The system calls [**Run**](https://docs.microsoft.com/uwp/api/windows.applicationmodel.background.ibackgroundtask.) method of a background task when the task is launched. Set the [**IBackgroundTask**](https://docs.microsoft.com/uwp/api/Windows.ApplicationModel.Background.IBackgroundTask) object passed into the method to the corresponding member variable. Register a handler for the [**Canceled**](https://docs.microsoft.com/uwp/api/windows.applicationmodel.background.ibackgroundtaskinstance.canceled) event, which will be raised if the system needs to shut down the background task. Then, set the [**Progress**](https://docs.microsoft.com/uwp/api/windows.applicationmodel.background.ibackgroundtaskinstance.progress) property to zero.

Next, call the background task object's [**GetDeferral**](https://docs.microsoft.com/uwp/api/windows.applicationmodel.background.ibackgroundtaskinstance.getdeferral) method to obtain a deferral. This tells the system not to shut down your task because you are performing asynchronous operations.

Next, call the helper method **TranscodeFileAsync**, which is defined in the next section. If that completes successfully, a helper method is called to launch a toast notification to alert the user that transcoding is complete.

At the end of the **Run** method, call [**Complete**](https://docs.microsoft.com/uwp/api/windows.applicationmodel.background.backgroundtaskdeferral.complete) on the deferral object to let the system know that your background task is complete and can be terminated.

[!code-cs[Run](./code/MediaProcessingTriggerWin10/cs/MediaProcessingBackgroundTask/MediaProcessingTask.cs#SnippetRun)]

In the **TranscodeFileAsync** helper method, the file names for the input and output files for the transcoding operations are retrieved from the [**LocalSettings**](https://docs.microsoft.com/uwp/api/windows.storage.applicationdata.localsettings) for your app. These values will be set by your foreground app. Create a [**StorageFile**](https://docs.microsoft.com/uwp/api/Windows.Storage.StorageFile) object for the input and output files and then create an encoding profile to use for transcoding.

Call [**PrepareFileTranscodeAsync**](https://docs.microsoft.com/uwp/api/windows.media.transcoding.mediatranscoder.preparefiletranscodeasync), passing in the input file, output file, and encoding profile. The [**PrepareTranscodeResult**](https://docs.microsoft.com/uwp/api/Windows.Media.Transcoding.PrepareTranscodeResult) object returned from this call lets you know if transcoding can be performed. If the [**CanTranscode**](https://docs.microsoft.com/uwp/api/windows.media.transcoding.preparetranscoderesult.cantranscode) property is true, call [**TranscodeAsync**](https://docs.microsoft.com/uwp/api/windows.media.transcoding.preparetranscoderesult.transcodeasync) to perform the transcoding operation.

The **AsTask** method enables you to track the progress the asynchronous operation or cancel it. Create a new **Progress** object, specifying the units of progress you desire and the name of the method that will be called to notify you of the current progress of the task. Pass the **Progress** object into the **AsTask** method along with the cancellation token that allows you to cancel the task.

[!code-cs[TranscodeFileAsync](./code/MediaProcessingTriggerWin10/cs/MediaProcessingBackgroundTask/MediaProcessingTask.cs#SnippetTranscodeFileAsync)]

In the method you used to create the Progress object in the previous step, **Progress**, set the progress of the background task instance. This will pass the progress to the foreground app, if it is running.

[!code-cs[Progress](./code/MediaProcessingTriggerWin10/cs/MediaProcessingBackgroundTask/MediaProcessingTask.cs#SnippetProgress)]

The **SendToastNotification** helper method creates a new toast notification by getting a template XML document for a toast that only has text content. The text element of the toast XML is set and then a new [**ToastNotification**](https://docs.microsoft.com/uwp/api/Windows.UI.Notifications.ToastNotification) object is created from the XML document. Finally, the toast is shown to the user by calling [**ToastNotifier.Show**](https://docs.microsoft.com/uwp/api/windows.ui.notifications.toastnotifier.show).

[!code-cs[SendToastNotification](./code/MediaProcessingTriggerWin10/cs/MediaProcessingBackgroundTask/MediaProcessingTask.cs#SnippetSendToastNotification)]

In the handler for the [**Canceled**](https://docs.microsoft.com/uwp/api/windows.applicationmodel.background.ibackgroundtaskinstance.canceled) event, which is called when the system cancels your background task, you can log the error for telemetry purposes.

[!code-cs[OnCanceled](./code/MediaProcessingTriggerWin10/cs/MediaProcessingBackgroundTask/MediaProcessingTask.cs#SnippetOnCanceled)]

## Register and launch the background task

Before you can launch the background task from your foreground app, you must update your foreground app's Package.appmanifest file to let the system know that your app uses a background task.

1.  In **Solution Explorer**, double-click the Package.appmanifest file icon to open the manifest editor.
2.  Select the **Declarations** tab.
3.  From **Available Declarations**, select **Background Tasks** and click **Add**.
4.  Under **Supported Declarations** make sure that the **Background Tasks** item is selected. Under **Properties**, select the checkbox for **Media processing**.
5.  In the **Entry Point** text box, specify the namespace and class name for your background test, separated by a period. For this example, the entry is:
   ```csharp
   MediaProcessingBackgroundTask.MediaProcessingTask
   ```
Next, you need to add a reference to your background task to your foreground app.
1.  In **Solution Explorer**, under your foreground app project, right-click the **References** folder and select **Add Reference...**.
2.  Expand the **Projects** node and select **Solution**.
3.  Check the box next to your background task project and click **OK**.

The rest of the code in this example should be added to your foreground app. First, you will need to add the following namespaces to your project.

[!code-cs[ForegroundUsing](./code/MediaProcessingTriggerWin10/cs/MediaProcessingTriggerWin10/MainPage.xaml.cs#SnippetForegroundUsing)]

Next, add the following member variables that are needed to register the background task.

[!code-cs[ForegroundMembers](./code/MediaProcessingTriggerWin10/cs/MediaProcessingTriggerWin10/MainPage.xaml.cs#SnippetForegroundMembers)]

The **PickFilesToTranscode** helper method uses a [**FileOpenPicker**](https://docs.microsoft.com/uwp/api/Windows.Storage.Pickers.FileOpenPicker) and a [**FileSavePicker**](https://docs.microsoft.com/uwp/api/Windows.Storage.Pickers.FileSavePicker) to open the input and output files for transcoding. The user may select files in a location that your app does not have access to. To make sure your background task can open the files, add them to the [**FutureAccessList**](https://docs.microsoft.com/uwp/api/windows.storage.accesscache.storageapplicationpermissions.futureaccesslist) for your app.

Finally, set entries for the input and output file names in the [**LocalSettings**](https://docs.microsoft.com/uwp/api/windows.storage.applicationdata.localsettings) for your app. The background task retrieves the file names from this location.

[!code-cs[PickFilesToTranscode](./code/MediaProcessingTriggerWin10/cs/MediaProcessingTriggerWin10/MainPage.xaml.cs#SnippetPickFilesToTranscode)]

To register the background task, create a new [**MediaProcessingTrigger**](https://docs.microsoft.com/uwp/api/Windows.ApplicationModel.Background.MediaProcessingTrigger) and a new [**BackgroundTaskBuilder**](https://docs.microsoft.com/uwp/api/Windows.ApplicationModel.Background.BackgroundTaskBuilder). Set the name of the background task builder so that you can identify it later. Set the [**TaskEntryPoint**](https://docs.microsoft.com/uwp/api/windows.applicationmodel.background.backgroundtaskbuilder.taskentrypoint) to the same namespace and class name string you used in the manifest file. Set the [**Trigger**](https://docs.microsoft.com/uwp/api/windows.applicationmodel.background.backgroundtaskregistration.trigger) property to the **MediaProcessingTrigger** instance.

Before registering the task, make sure you unregister any previously registered tasks by looping through the [**AllTasks**](https://docs.microsoft.com/uwp/api/windows.applicationmodel.background.backgroundtaskregistration.alltasks) collection and calling [**Unregister**](https://docs.microsoft.com/uwp/api/windows.applicationmodel.background.ibackgroundtaskregistration.unregister) on any tasks that have the name you specified in the [**BackgroundTaskBuilder.Name**](https://docs.microsoft.com/uwp/api/windows.applicationmodel.background.backgroundtaskbuilder.name) property.

Register the background task by calling [**Register**](https://docs.microsoft.com/uwp/api/windows.applicationmodel.background.backgroundtaskbuilder.register). Register handlers for the [**Completed**](https://docs.microsoft.com/uwp/api/windows.applicationmodel.background.backgroundtaskregistration.completed) and [**Progress**](https://docs.microsoft.com/uwp/api/windows.applicationmodel.background.ibackgroundtaskregistration.progress) events.

[!code-cs[RegisterBackgroundTask](./code/MediaProcessingTriggerWin10/cs/MediaProcessingTriggerWin10/MainPage.xaml.cs#SnippetRegisterBackgroundTask)]

A typical app will register their background task when the app is initially launched, such as in the **OnNavigatedTo** event.

Launch the background task by calling the **MediaProcessingTrigger** object's [**RequestAsync**](https://docs.microsoft.com/uwp/api/windows.applicationmodel.background.mediaprocessingtrigger.requestasync) method. The [**MediaProcessingTriggerResult**](https://docs.microsoft.com/uwp/api/Windows.ApplicationModel.Background.MediaProcessingTriggerResult) object returned by this method lets you know whether the background task was started successfully, and if not, lets you know why the background task wasn't launched. 

[!code-cs[LaunchBackgroundTask](./code/MediaProcessingTriggerWin10/cs/MediaProcessingTriggerWin10/MainPage.xaml.cs#SnippetLaunchBackgroundTask)]

A typical app will launch the background task in response to user interaction, such as in the **Click** event of a UI control.

The **OnProgress** event handler is called when the background task updates the progress of the operation. You can use this opportunity to update your UI with progress information.

[!code-cs[OnProgress](./code/MediaProcessingTriggerWin10/cs/MediaProcessingTriggerWin10/MainPage.xaml.cs#SnippetOnProgress)]

The **OnCompleted** event handler is called when the background task has finished running. This is another opportunity to update your UI to give status information to the user.

[!code-cs[OnCompleted](./code/MediaProcessingTriggerWin10/cs/MediaProcessingTriggerWin10/MainPage.xaml.cs#SnippetOnCompleted)]


 

 




