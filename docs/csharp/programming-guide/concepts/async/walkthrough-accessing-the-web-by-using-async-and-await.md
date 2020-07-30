---
title: 演练：使用 async 和 await 访问 Web (C#)
ms.date: 07/20/2015
ms.assetid: c95d8d71-5a98-4bf0-aaf4-45fed2ebbacd
ms.openlocfilehash: edc45f252b75c742afe4f073ca7dd92d9784d2c4
ms.sourcegitcommit: a241301495a84cc8c64fe972330d16edd619868b
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/01/2020
ms.locfileid: "84240642"
---
# <a name="walkthrough-accessing-the-web-by-using-async-and-await-c"></a>演练：使用 async 和 await 访问 Web (C#)

使用 async/await 功能可以更轻松直观地编写异步程序。 你可以编写类似于同步代码的异步代码，并让编译器处理异步代码通常需要的疑难回调函数和延续。

有关 Async 功能的详细信息，请参阅[使用 Async 和 Await 的异步编程 (C#)](./index.md)。

本演练从对网站列表中的字节数进行求和的同步 Windows Presentation Foundation (WPF) 应用程序入手， 然后使用新功能将该应用程序转换为异步解决方案。

如果不想自行生成应用，可以下载[异步示例：访问 Web 演练（C# 和 Visual Basic）](https://code.msdn.microsoft.com/Async-Sample-Accessing-the-9c10497f)。

> [!NOTE]
> 若要运行该示例，计算机上必须安装有 Visual Studio 2012 或更高版本和 .NET Framework 4.5 或更高版本。

## <a name="create-a-wpf-application"></a>创建 WPF 应用程序

1. 启动 Visual Studio。

2. 在菜单栏上，依次选择“文件” > “新建” > “项目”。

     **“新建项目”** 对话框随即打开。

3. 在“已安装的模板”窗格中，选择“Visual C#”，然后从项目类型列表选择“WPF 应用程序”。

4. 在“名称”文本框中，输入 `AsyncExampleWPF`，然后选择“确定”按钮。

     新项目将出现在“解决方案资源管理器”中。

## <a name="design-a-simple-wpf-mainwindow"></a>设计简单的 WPF MainWindow

1. 在 Visual Studio 代码编辑器中，选择 **“MainWindow.xaml”** 选项卡。

2. 如果“工具箱”窗口不可见，则打开“视图”菜单，然后选择“工具箱”  。

3. 向“MainWindow”窗口添加一个“Button”控件和一个“TextBox”控件。

4. 突出显示“TextBox”控件，在“属性”窗口中，设置下列值：

    - 将“名称”属性设置为 `resultsTextBox`。

    - 将“高度”属性设置为 250。

    - 将“宽度”属性设置为 500。

    - 在“文本”选项卡中，指定等宽字体，例如 Lucida Console 或 Global Monospace。

5. 突出显示“Button”控件，在“属性”窗口中，设置下列值：

    - 将“名称”属性设置为 `startButton`。

    - 将“内容”属性的值从“Button”更改为“Start”。

6. 确定文本框和按钮的位置，以便它们都在“MainWindow”窗口中显示。

     有关 WPF XAML 设计器的详细信息，请参阅[使用 XAML 设计器创建 UI](/visualstudio/xaml-tools/creating-a-ui-by-using-xaml-designer-in-visual-studio)。

## <a name="add-a-reference"></a>添加引用

1. 在“解决方案资源管理器”中，突出显示项目的名称。

2. 在菜单栏上，选择“项目” > “添加引用”。

     此时将显示“引用管理器”对话框。

3. 在对话框顶部，验证项目是否以 .NET Framework 4.5 或更高版本为目标。

4. 在“程序集”类别中，选择“Framework”（如果尚未选择它） 。

5. 在名称列表中，选中“System.Net.Http”复选框。

6. 选择“确定”按钮关闭对话框。

## <a name="add-necessary-using-directives"></a>添加必要的 using 指令

1. 在“解决方案资源管理器”中，打开 MainWindow.xaml.cs 的快捷菜单，然后选择“查看代码”。

2. 将下列 `using` 指令添加到代码文件的顶部（如果它们尚不存在）。

    ```csharp
    using System.Net.Http;
    using System.Net;
    using System.IO;
    ```

## <a name="create-a-synchronous-app"></a>创建同步应用

1. 在设计窗口 MainWindow.xaml 中，双击“启动”按钮以在 MainWindow.xaml.cs 中创建 `startButton_Click` 事件处理程序。

2. 在 MainWindow.xaml.cs 中，将下列代码复制到 `startButton_Click` 的正文中：

    ```csharp
    resultsTextBox.Clear();
    SumPageSizes();
    resultsTextBox.Text += "\r\nControl returned to startButton_Click.";
    ```

    代码调用驱动应用程序 `SumPageSizes` 的方法，并在控件返回到 `startButton_Click` 时显示一条消息。

3. 该同步解决方案的代码包含以下四个方法：

    - `SumPageSizes`，从 `SetUpURLList` 获取网页 URL 列表并随后调用 `GetURLContents` 和 `DisplayResults` 以处理每个 URL。

    - `SetUpURLList`，生成并返回 Web 地址列表。

    - `GetURLContents`，下载每个网站的内容并将内容作为字节数组返回。

    - `DisplayResults`，显示每个 URL 的字节数组中的字节数。

    复制以下四个方法，然后将它们粘贴在 MainWindow.xaml.cs 中的 `startButton_Click` 事件处理程序下：

    ```csharp
    private void SumPageSizes()
    {
        // Make a list of web addresses.
        List<string> urlList = SetUpURLList();

        var total = 0;
        foreach (var url in urlList)
        {
            // GetURLContents returns the contents of url as a byte array.
            byte[] urlContents = GetURLContents(url);

            DisplayResults(url, urlContents);

            // Update the total.
            total += urlContents.Length;
        }

        // Display the total count for all of the web addresses.
        resultsTextBox.Text += $"\r\n\r\nTotal bytes returned:  {total}\r\n";
    }

    private List<string> SetUpURLList()
    {
        var urls = new List<string>
        {
            "https://msdn.microsoft.com/library/windows/apps/br211380.aspx",
            "https://msdn.microsoft.com",
            "https://msdn.microsoft.com/library/hh290136.aspx",
            "https://msdn.microsoft.com/library/ee256749.aspx",
            "https://msdn.microsoft.com/library/hh290138.aspx",
            "https://msdn.microsoft.com/library/hh290140.aspx",
            "https://msdn.microsoft.com/library/dd470362.aspx",
            "https://msdn.microsoft.com/library/aa578028.aspx",
            "https://msdn.microsoft.com/library/ms404677.aspx",
            "https://msdn.microsoft.com/library/ff730837.aspx"
        };
        return urls;
    }

    private byte[] GetURLContents(string url)
    {
        // The downloaded resource ends up in the variable named content.
        var content = new MemoryStream();

        // Initialize an HttpWebRequest for the current URL.
        var webReq = (HttpWebRequest)WebRequest.Create(url);
        //add header prvent 403 error
        webReq.ContentType = "application/json; charset=utf-8";
        webReq.UserAgent =
            "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/84.0.4147.89 Safari/537.36 Edg/84.0.522.44"
        // Send the request to the Internet resource and wait for
        // the response.
        // Note: you can't use HttpWebRequest.GetResponse in a Windows Store app.
        using (WebResponse response = webReq.GetResponse())
        {
            // Get the data stream that is associated with the specified URL.
            using (Stream responseStream = response.GetResponseStream())
            {
                // Read the bytes in responseStream and copy them to content.
                responseStream.CopyTo(content);
            }
        }

        // Return the result as a byte array.
        return content.ToArray();
    }

    private void DisplayResults(string url, byte[] content)
    {
        // Display the length of each website. The string format
        // is designed to be used with a monospaced font, such as
        // Lucida Console or Global Monospace.
        var bytes = content.Length;
        // Strip off the "https://".
        var displayURL = url.Replace("https://", "");
        resultsTextBox.Text += $"\n{displayURL,-58} {bytes,8}";
    }
    ```

## <a name="test-the-synchronous-solution"></a>测试同步解决方案

按 F5 键运行程序，然后选择“启动”按钮。

此时应显示类似于以下列表的输出：

```text
msdn.microsoft.com/library/windows/apps/br211380.aspx        383832
msdn.microsoft.com                                            33964
msdn.microsoft.com/library/hh290136.aspx               225793
msdn.microsoft.com/library/ee256749.aspx               143577
msdn.microsoft.com/library/hh290138.aspx               237372
msdn.microsoft.com/library/hh290140.aspx               128279
msdn.microsoft.com/library/dd470362.aspx               157649
msdn.microsoft.com/library/aa578028.aspx               204457
msdn.microsoft.com/library/ms404677.aspx               176405
msdn.microsoft.com/library/ff730837.aspx               143474

Total bytes returned:  1834802

Control returned to startButton_Click.
```

请注意，显示计数需要几秒钟时间。 与此同时，在等待请求的资源下载时，UI 线程处于被阻止状态。 因此，选择“启动”按钮后，将无法移动、最大化、最小化显示窗口，甚至也无法关闭显示窗口。 在字节计数开始显示之前，这些操作都会失败。 如果网站没有响应，将不会指示哪个网站失败。 甚至停止等待和关闭程序也会很困难。

## <a name="convert-geturlcontents-to-an-asynchronous-method"></a>将 GetURLContents 转换为异步方法

1. 要将同步解决方案转换为异步解决方案，最佳着手点在 `GetURLContents` 中，因为对 <xref:System.Net.HttpWebRequest> 方法 <xref:System.Net.HttpWebRequest.GetResponse%2A> 的调用以及对 <xref:System.IO.Stream> 方法 <xref:System.IO.Stream.CopyTo%2A> 的调用是应用程序访问 Web 的位置。 .NET Framework 提供两种方法的异步版本，使转换变得轻松。

     有关 `GetURLContents` 中使用的方法的详细信息，请参阅 <xref:System.Net.WebRequest>。

    > [!NOTE]
    > 在你按照本演练中的步骤进行操作的过程中，将出现多个编译器错误。 你可以忽略这些错误并继续演练。

     将在 `GetURLContents` 的第三行中调用的方法从 `GetResponse` 更改为基于任务的异步 <xref:System.Net.WebRequest.GetResponseAsync%2A> 方法。

    ```csharp
    using (WebResponse response = webReq.GetResponseAsync())
    ```

2. `GetResponseAsync` 返回 <xref:System.Threading.Tasks.Task%601>。 在这种情况下，任务返回变量 `TResult` 具有类型 <xref:System.Net.WebResponse>。 该任务是在请求的任务已下载且任务已完成运行后，生成实际 `WebResponse` 对象的承诺。

     要从任务检索 `WebResponse` 值，请将 [await](../../../language-reference/operators/await.md) 运算符应用到对 `GetResponseAsync` 的调用，如下列代码所示。

    ```csharp
    using (WebResponse response = await webReq.GetResponseAsync())
    ```

     `await` 运算符将当前方法 `GetURLContents` 的执行挂起，直到完成等待的任务为止。 同时，控制权返回给当前方法的调用方。 在此示例中，当前方法是 `GetURLContents`，调用方是 `SumPageSizes`。 任务完成时，承诺的 `WebResponse` 对象作为等待的任务的值生成，并分配给变量 `response`。

     上一条语句可以分为以下两条语句，以阐明所发生的情况。

    ```csharp
    //Task<WebResponse> responseTask = webReq.GetResponseAsync();
    //using (WebResponse response = await responseTask)
    ```

     对 `webReq.GetResponseAsync` 的调用返回 `Task(Of WebResponse)` 或 `Task<WebResponse>`。 然后，await 运算符将应用于任务以检索 `WebResponse` 值。

     如果你的异步方法需要完成不依赖于任务的完成的工作，则在调用异步方法之后及应用 `await` 运算符之前的这段时间，该方法可以在这两个语句之间继续完成该工作。 有关示例，请参阅[如何使用 Async 和 Await 并行发起多个 Web 请求 (C#)](./how-to-make-multiple-web-requests-in-parallel-by-using-async-and-await.md) 和[如何使用 Task.WhenAll 扩展异步演练 (C#)](./how-to-extend-the-async-walkthrough-by-using-task-whenall.md)。

3. 因为在上一步中添加了 `await` 运算符，所以会发生编译器错误。 该运算符仅可在使用 [async](../../../language-reference/keywords/async.md) 修饰符标记的方法中使用。 当你重复转换步骤以使用对 `CopyToAsync` 的调用替换对 `CopyTo` 的调用时，请忽略该错误。

    - 更改被调用到 <xref:System.IO.Stream.CopyToAsync%2A> 的方法的名称。

    - `CopyTo` 或 `CopyToAsync` 方法复制字节到其参数 `content`，并且不返回有意义的值。 在同步版本中，对 `CopyTo` 的调用是不返回值的简单语句。 异步版本 `CopyToAsync` 返回 <xref:System.Threading.Tasks.Task>。 任务函数类似“Task(void)”，并让该方法能够等待。 应用 `Await` 或 `await` 到对 `CopyToAsync` 的调用，如下列代码所示。

        ```csharp
        await responseStream.CopyToAsync(content);
        ```

         上一条语句缩写以下两行代码。

        ```csharp
        // CopyToAsync returns a Task, not a Task<T>.
        //Task copyTask = responseStream.CopyToAsync(content);

        // When copyTask is completed, content contains a copy of
        // responseStream.
        //await copyTask;
        ```

4. `GetURLContents` 中仍需要完成的操作是调整方法签名。 仅可在使用 [async](../../../language-reference/keywords/async.md) 修饰符标记的方法中使用 `await` 运算符。 添加修饰符以将方法标记为*异步方法*，如下列代码所示。

    ```csharp
    private async byte[] GetURLContents(string url)
    ```

5. 异步方法的返回类型在 C# 中只能为 <xref:System.Threading.Tasks.Task>、<xref:System.Threading.Tasks.Task%601> 或 `void`。 通常情况下，`void` 的返回类型仅可在异步事件处理程序中使用，其中需要 `void`。 在其他情况下，如果完成的方法具有返回 T 类型的值的 [return](../../../language-reference/keywords/return.md) 语句，则使用 `Task(T)`；如果完成的方法不返回有意义的值，则使用 `Task`。 可以将 `Task` 返回类型视为表示“Task(void)”。

     有关详细信息，请参阅[异步返回类型 (C#)](./async-return-types.md)。

     方法 `GetURLContents` 具有 return 语句，且该语句返回字节数组。 因此，异步版本的返回类型为 Task(T)，其中 T 为字节数组。 在方法签名中进行下列更改：

    - 将返回类型更改为 `Task<byte[]>`。

    - 按照约定，异步方法的名称以“Async”结尾，因此，请重命名方法 `GetURLContentsAsync`。

     下列代码显示这些更改。

    ```csharp
    private async Task<byte[]> GetURLContentsAsync(string url)
    ```

     进行这几处更改后，`GetURLContents` 到异步方法的转换完成。

## <a name="convert-sumpagesizes-to-an-asynchronous-method"></a>将 SumPageSizes 转换为异步方法

1. 为 `SumPageSizes` 重复之前过程中的步骤。 首先，将对 `GetURLContents` 的调用更改为异步调用。

    - 将调用的方法的名称从 `GetURLContents` 更改为 `GetURLContentsAsync`（如果尚未执行此操作）。

    - 将 `await` 应用到 `GetURLContentsAsync` 返回的任务，以便获取字节数组值。

     下列代码显示这些更改。

    ```csharp
    byte[] urlContents = await GetURLContentsAsync(url);
    ```

     上一个分配缩写以下两行代码。

    ```csharp
    // GetURLContentsAsync returns a Task<T>. At completion, the task
    // produces a byte array.
    //Task<byte[]> getContentsTask = GetURLContentsAsync(url);
    //byte[] urlContents = await getContentsTask;
    ```

2. 在方法签名中进行下列更改：

    - 使用 `async` 修饰符标记方法。

    - 将“Async”添加到方法名称。

    - 这一次没有任务返回变量 T，因为 `SumPageSizesAsync` 不返回 T 的值。（该方法没有 `return` 语句。）但是，该方法必须返回 `Task` 才能进行等待。 因此，将该方法的返回类型从 `void` 更改为 `Task`。

    下列代码显示这些更改。

    ```csharp
    private async Task SumPageSizesAsync()
    ```

     从 `SumPageSizes` 到 `SumPageSizesAsync` 的转换完成。

## <a name="convert-startbutton_click-to-an-asynchronous-method"></a>将 startButton_Click 转换为异步方法

1. 在事件处理程序中，将调用的方法的名称从 `SumPageSizes` 更改为 `SumPageSizesAsync`（如果尚未执行此操作）。

2. 由于 `SumPageSizesAsync` 是异步方法，请更改事件处理程序中的代码以等待结果。

     对 `SumPageSizesAsync` 的调用反射对 `GetURLContentsAsync` 中的 `CopyToAsync` 的调用。 调用返回的是 `Task`，而不是 `Task(T)`。

     与之前的过程一样，你可以使用一条语句或两条语句转换调用。 下列代码显示这些更改。

    ```csharp
    // One-step async call.
    await SumPageSizesAsync();

    // Two-step async call.
    //Task sumTask = SumPageSizesAsync();
    //await sumTask;
    ```

3. 要防止意外重新进入操作，请在 `startButton_Click` 顶部添加下列语句，以禁用“启动”按钮。

    ```csharp
    // Disable the button until the operation is complete.
    startButton.IsEnabled = false;
    ```

     可以重新启用事件处理程序末尾的按钮。

    ```csharp
    // Reenable the button in case you want to run the operation again.
    startButton.IsEnabled = true;
    ```

     有关重新进入的详细信息，请参阅[处理异步应用中的重新进入 (C#)](./handling-reentrancy-in-async-apps.md)。

4. 最后，将 `async` 修饰符添加到声明，以便事件处理程序可以等待 `SumPagSizesAsync`。

    ```csharp
    private async void startButton_Click(object sender, RoutedEventArgs e)
    ```

     通常情况下，事件处理程序的名称不会更改。 返回类型没有更改为 `Task`，因为事件处理程序必须返回 `void`。

     项目从同步处理到异步处理的转换完成。

## <a name="test-the-asynchronous-solution"></a>测试异步解决方案

1. 按 F5 键运行程序，然后选择“启动”按钮。

2. 此时应显示类似于同步解决方案的输出的输出。 但是，请注意下列差异。

    - 处理完成后，所有结果不会同时出现。 例如，两个程序都在 `startButton_Click` 中包含一行可以清除文本框的代码。 目的在于，在一组结果显示后，第二次选择“启动”按钮时，可以清除运行之间的文本框。 在同步版本中，下载完成且 UI 线程可以自由完成其他工作时，文本框在计数第二次显示之前即被清除。 在异步版本中，选择“启动”按钮后立即清除文本框。

    - 最重要的是，UI 线程在下载过程中未被阻止。 在 Web 资源下载、计数和显示期间，可以移动窗口或调整窗口大小。 如果其中一个网站速度缓慢或没有响应，则可以通过选择“关闭”按钮取消操作（右上角红色字段中的 x）。

## <a name="replace-method-geturlcontentsasync-with-a-net-method"></a>使用 .NET 方法替换方法 GetURLContentsAsync

1. .NET Framework 4.5 及更高版本提供了许多可供使用的异步方法。 其中一个是 <xref:System.Net.Http.HttpClient> 方法 <xref:System.Net.Http.HttpClient.GetByteArrayAsync%28System.String%29>，它可以执行本演练所需的操作。 你可以使用它来替代你在早前过程中创建的 `GetURLContentsAsync` 方法。

     第一步是在方法 `SumPageSizesAsync` 中创建 `HttpClient` 对象。 在方法的开头添加下列声明。

    ```csharp
    // Declare an HttpClient object and increase the buffer size. The
    // default buffer size is 65,536.
    HttpClient client =
        new HttpClient() { MaxResponseContentBufferSize = 1000000 };
    ```

2. 在 `SumPageSizesAsync,` 中，使用对 `HttpClient` 方法的调用替换对 `GetURLContentsAsync` 方法的调用。

    ```csharp
    byte[] urlContents = await client.GetByteArrayAsync(url);
    ```

3. 删除或注释禁用你编写的 `GetURLContentsAsync` 方法。

4. 按 F5 键运行程序，然后选择“启动”按钮。

     此版本的项目的行为应与“测试异步解决方案”过程描述的行为匹配，且你的工作量应该更少。

## <a name="example-code"></a>示例代码

下列代码包含使用你编写的异步 `GetURLContentsAsync` 方法从同步解决方案转换为异步解决方案的完整示例。 请注意，它与原始同步解决方案十分类似。

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using System.Windows;
using System.Windows.Controls;
using System.Windows.Data;
using System.Windows.Documents;
using System.Windows.Input;
using System.Windows.Media;
using System.Windows.Media.Imaging;
using System.Windows.Navigation;
using System.Windows.Shapes;

// Add the following using directives, and add a reference for System.Net.Http.
using System.Net.Http;
using System.IO;
using System.Net;

namespace AsyncExampleWPF
{
    public partial class MainWindow : Window
    {
        public MainWindow()
        {
            InitializeComponent();
        }

        private async void startButton_Click(object sender, RoutedEventArgs e)
        {
            // Disable the button until the operation is complete.
            startButton.IsEnabled = false;

            resultsTextBox.Clear();

            // One-step async call.
            await SumPageSizesAsync();

            // Two-step async call.
            //Task sumTask = SumPageSizesAsync();
            //await sumTask;

            resultsTextBox.Text += "\r\nControl returned to startButton_Click.\r\n";

            // Reenable the button in case you want to run the operation again.
            startButton.IsEnabled = true;
        }

        private async Task SumPageSizesAsync()
        {
            // Make a list of web addresses.
            List<string> urlList = SetUpURLList();

            var total = 0;

            foreach (var url in urlList)
            {
                byte[] urlContents = await GetURLContentsAsync(url);

                // The previous line abbreviates the following two assignment statements.

                // GetURLContentsAsync returns a Task<T>. At completion, the task
                // produces a byte array.
                //Task<byte[]> getContentsTask = GetURLContentsAsync(url);
                //byte[] urlContents = await getContentsTask;

                DisplayResults(url, urlContents);

                // Update the total.
                total += urlContents.Length;
            }
            // Display the total count for all of the websites.
            resultsTextBox.Text +=
                $"\r\n\r\nTotal bytes returned:  {total}\r\n";
        }

        private List<string> SetUpURLList()
        {
            List<string> urls = new List<string>
            {
                "https://msdn.microsoft.com/library/windows/apps/br211380.aspx",
                "https://msdn.microsoft.com",
                "https://msdn.microsoft.com/library/hh290136.aspx",
                "https://msdn.microsoft.com/library/ee256749.aspx",
                "https://msdn.microsoft.com/library/hh290138.aspx",
                "https://msdn.microsoft.com/library/hh290140.aspx",
                "https://msdn.microsoft.com/library/dd470362.aspx",
                "https://msdn.microsoft.com/library/aa578028.aspx",
                "https://msdn.microsoft.com/library/ms404677.aspx",
                "https://msdn.microsoft.com/library/ff730837.aspx"
            };
            return urls;
        }

        private async Task<byte[]> GetURLContentsAsync(string url)
        {
            // The downloaded resource ends up in the variable named content.
            var content = new MemoryStream();

            // Initialize an HttpWebRequest for the current URL.
            var webReq = (HttpWebRequest)WebRequest.Create(url);

            // Send the request to the Internet resource and wait for
            // the response.
            using (WebResponse response = await webReq.GetResponseAsync())

            // The previous statement abbreviates the following two statements.

            //Task<WebResponse> responseTask = webReq.GetResponseAsync();
            //using (WebResponse response = await responseTask)
            {
                // Get the data stream that is associated with the specified url.
                using (Stream responseStream = response.GetResponseStream())
                {
                    // Read the bytes in responseStream and copy them to content.
                    await responseStream.CopyToAsync(content);

                    // The previous statement abbreviates the following two statements.

                    // CopyToAsync returns a Task, not a Task<T>.
                    //Task copyTask = responseStream.CopyToAsync(content);

                    // When copyTask is completed, content contains a copy of
                    // responseStream.
                    //await copyTask;
                }
            }
            // Return the result as a byte array.
            return content.ToArray();
        }

        private void DisplayResults(string url, byte[] content)
        {
            // Display the length of each website. The string format
            // is designed to be used with a monospaced font, such as
            // Lucida Console or Global Monospace.
            var bytes = content.Length;
            // Strip off the "https://".
            var displayURL = url.Replace("https://", "");
            resultsTextBox.Text += $"\n{displayURL,-58} {bytes,8}";
        }
    }
}
```

下列代码包含使用 `HttpClient` 方法 `GetByteArrayAsync` 的解决方案的完整示例。

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using System.Windows;
using System.Windows.Controls;
using System.Windows.Data;
using System.Windows.Documents;
using System.Windows.Input;
using System.Windows.Media;
using System.Windows.Media.Imaging;
using System.Windows.Navigation;
using System.Windows.Shapes;

// Add the following using directives, and add a reference for System.Net.Http.
using System.Net.Http;
using System.IO;
using System.Net;

namespace AsyncExampleWPF
{
    public partial class MainWindow : Window
    {
        public MainWindow()
        {
            InitializeComponent();
        }

        private async void startButton_Click(object sender, RoutedEventArgs e)
        {
            resultsTextBox.Clear();

            // Disable the button until the operation is complete.
            startButton.IsEnabled = false;

            // One-step async call.
            await SumPageSizesAsync();

            //// Two-step async call.
            //Task sumTask = SumPageSizesAsync();
            //await sumTask;

            resultsTextBox.Text += "\r\nControl returned to startButton_Click.\r\n";

            // Reenable the button in case you want to run the operation again.
            startButton.IsEnabled = true;
        }

        private async Task SumPageSizesAsync()
        {
            // Declare an HttpClient object and increase the buffer size. The
            // default buffer size is 65,536.
            HttpClient client =
                new HttpClient() { MaxResponseContentBufferSize = 1000000 };

            // Make a list of web addresses.
            List<string> urlList = SetUpURLList();

            var total = 0;

            foreach (var url in urlList)
            {
                // GetByteArrayAsync returns a task. At completion, the task
                // produces a byte array.
                byte[] urlContents = await client.GetByteArrayAsync(url);

                // The following two lines can replace the previous assignment statement.
                //Task<byte[]> getContentsTask = client.GetByteArrayAsync(url);
                //byte[] urlContents = await getContentsTask;

                DisplayResults(url, urlContents);

                // Update the total.
                total += urlContents.Length;
            }

            // Display the total count for all of the websites.
            resultsTextBox.Text +=
                $"\r\n\r\nTotal bytes returned:  {total}\r\n";
        }

        private List<string> SetUpURLList()
        {
            List<string> urls = new List<string>
            {
                "https://msdn.microsoft.com/library/windows/apps/br211380.aspx",
                "https://msdn.microsoft.com",
                "https://msdn.microsoft.com/library/hh290136.aspx",
                "https://msdn.microsoft.com/library/ee256749.aspx",
                "https://msdn.microsoft.com/library/hh290138.aspx",
                "https://msdn.microsoft.com/library/hh290140.aspx",
                "https://msdn.microsoft.com/library/dd470362.aspx",
                "https://msdn.microsoft.com/library/aa578028.aspx",
                "https://msdn.microsoft.com/library/ms404677.aspx",
                "https://msdn.microsoft.com/library/ff730837.aspx"
            };
            return urls;
        }

        private void DisplayResults(string url, byte[] content)
        {
            // Display the length of each website. The string format
            // is designed to be used with a monospaced font, such as
            // Lucida Console or Global Monospace.
            var bytes = content.Length;
            // Strip off the "https://".
            var displayURL = url.Replace("https://", "");
            resultsTextBox.Text += $"\n{displayURL,-58} {bytes,8}";
        }
    }
}
```

## <a name="see-also"></a>请参阅

- [异步示例：访问 Web 演练（C# 和 Visual Basic）](https://docs.microsoft.com/previous-versions/visualstudio/visual-studio-2012/hh300224(v=vs.110))
- [async](../../../language-reference/keywords/async.md)
- [await](../../../language-reference/operators/await.md)
- [使用 Async 和 Await 的异步编程 (C#)](./index.md)
- [异步返回类型 (C#)](./async-return-types.md)
- [基于任务的异步编程 (TAP)](https://www.microsoft.com/download/details.aspx?id=19957)
- [如何使用 Task.WhenAll 扩展异步演练 (C#)](./how-to-extend-the-async-walkthrough-by-using-task-whenall.md)
- [如何使用 Async 和 Await 并行发出多个 Web 请求 (C#)](./how-to-make-multiple-web-requests-in-parallel-by-using-async-and-await.md)
