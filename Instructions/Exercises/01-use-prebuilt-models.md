---
lab:
    title: 'Use prebuilt Document Intelligence models'
    module: 'Module 11 - Reading Text in Images and Documents'
---

# Use prebuilt Document Intelligence models

In this exercise, you'll set up an Azure AI Document Intelligence resource in your Azure subscription. You'll use both the Azure AI Document Intelligence Studio and C# code to submit forms to that resource for analysis.

## Create an Azure AI Document Intelligence resource

Before you can call the Azure AI Document Intelligence service, you must create a resource to host that service in Azure:

1. In the [Azure portal](https://portal.azure.com/learn.docs.microsoft.com?azure-portal=true), on the home page in the top search box, type **Document intelligence** and then press <kbd>Enter</kbd>.
1. On the **Document Intelligence** page, select **Create**.
1. On the **Basics** page of **Create Document Intelligence** wizard, in the **Subscription** list, select your subscription.
1. Under the **Resource group** list, select **Create new**.
1. In the **Name** textbox, type **DocIntelligenceResources** and then select **OK**.
1. In the **Region** list, select a region near you.
1. In the **Name** box, type a globally unique name for your resource.
1. In the **Pricing tier** list, select **Free F0** and then select **Review + create**. If you don't have a Free tier available, select **Standard S0**
1. On the **Review + create** page, select **Create**, and then wait while Azure creates the Azure AI Document Intelligence resource.
1. When the deployment is complete, select **Go to resource**. Keep this page open for the rest of this exercise.

## Use the read model

Let's start by using the **Azure AI Document Intelligence Studio** and the read model to analyze a document with multiple languages. You'll connect Azure AI Document Intelligence Studio to the resource you just created to perform the analysis:

1. Open a new browser tab and go to [Azure AI Document Intelligence Studio](https://formrecognizer.appliedai.azure.com/studio).
1. Under **Document Analysis**, select **Read**.
1. If you are asked to sign into your account, use your Azure credentials.
1. If you are asked which Azure AI Document Intelligence resource to use, select the subscription and resource name you used when you created the Azure AI Document Intelligence resource.
1. In the list of documents on the left, select **read-german.png**.

    ![Screenshot showing the Read page in Azure AI Document Intelligence Studio.](../media/read-german-sample.png#lightbox)

1. At the top-left, select **Analyze**.
1. When the analysis is complete, the text extracted from the image is shown on the right in the **Content** tab. Review this text and compare it to the text in the original image for accuracy.
1. Select the **Result** tab. This tab displays the extracted JSON code. 
1. Scroll to the bottom of the JSON code in the **Result** tab. Notice that the read model has detected the language of each span. Most spans are in German (language code `de`) but the last span is in English (language code `en`).

    ![Screenshot showing the detection of language for two spans in the results from the read model in Azure AI Document Intelligence Studio.](../media/language-detection.png#lightbox)

## Prepare to develop an app in Visual Studio Code

Now let's explore the app that uses the Azure Document Intelligence service SDK. You'll develop your app using Visual Studio Code. The code files for your app have been provided in a GitHub repo.

> **Tip**: If you have already cloned the **mslearn-ai-document-intelligence** repo, open it in Visual Studio code. Otherwise, follow these steps to clone it to your development environment.

1. Start Visual Studio Code.
1. Open the palette (SHIFT+CTRL+P) and run a **Git: Clone** command to clone the `https://github.com/MicrosoftLearning/mslearn-ai-document-intelligence` repository to a local folder (it doesn't matter which folder).
1. When the repository has been cloned, open the folder in Visual Studio Code.
1. Wait while additional files are installed to support the C# code projects in the repo.

    > **Note**: If you are prompted to add required assets to build and debug, select **Not Now**. If you are prompted with the Message *Detected an Azure Function Project in folder*, you can safely close that message.

## Configure your application

Applications for both C# and Python have been provided, as well as a sample pdf file you'll use to test the document intelligence. Both apps feature the same functionality. First, you'll complete some key parts of the application to enable using your Azure Document Intelligence resource.

1. Examine the following invoice and note some of its fields and values. This is the invoice that your code will analyze.

    ![Screenshot showing a sample invoice document.](../media/sample-invoice.png#lightbox)

1. In Visual Studio Code, in the **Explorer** pane, browse to the **Labfiles/01-prebuild-models** folder and expand the **CSharp** or **Python** folder depending on your language preference. Each folder contains the language-specific files for an app into which you're you're going to integrate Azure OpenAI functionality.

1. Right-click the **CSharp** or **Python** folder containing your code files and open an integrated terminal. Then install the Azure Form Recognizer SDK package by running the appropriate command for your language preference:

    **C#**:

    ```powershell
    dotnet add package Azure.AI.FormRecognizer --version 4.1.0
    ```

    **Python**:

    ```powershell
    pip install azure-ai-formrecognizer==3.3.0
    ```

## Add code to use the Azure OpenAI service

Now you're ready to use the Azure Form Recognizer SDK to evaluate the pdf file.

1. Switch to the browser tab that displays the Azure AI Document Intelligence overview in the Azure portal. To the right of the **Endpoint** value, click the **Copy to clipboard** button.
1. In the **Explorer** pane, in the **CSharp** or **Python** folder, open the code file for your preferred language, and replace `<Endpoint URL>` with the string you just copied:

    **C#**: ***Program.cs***

    ```csharp
    string endpoint = "<Endpoint URL>";
    ```

    **Python**: ***document-analysis.py***

    ```python
    endpoint = "<Endpoint URL>"
    ```

1. Switch to the browser tab that displays the Azure AI Document Intelligence overview in the Azure portal. To the right of the **KEY 1** value, click the *Copy to clipboard** button.
1. In the code file in Visual Studio Code, locate this line and replace `<API Key>` with the string you just copied:

    **C#**

    ```csharp
    string apiKey = "<API Key>";
    ```

    **Python**

    ```python
    key = "<API Key>"
    ```

1. Locate the comment `Create the client`. Following that, on new lines, enter the following code:

    **C#**

    ```csharp
    var cred = new AzureKeyCredential(apiKey);
    var client = new DocumentAnalysisClient(new Uri(endpoint), cred);
    ```

    **Python**

    ```python
    document_analysis_client = DocumentAnalysisClient(
        endpoint=endpoint, credential=AzureKeyCredential(key)
    )
    ```

1. Locate the comment `Analyze the invoice`. Following that, on new lines, enter the following code:

    **C#**

    ```csharp
    AnalyzeDocumentOperation operation = await client.AnalyzeDocumentFromUriAsync(WaitUntil.Completed, "prebuilt-invoice", fileUri);
    await operation.WaitForCompletionAsync();
    ```

    **Python**

    ```python
    poller = document_analysis_client.begin_analyze_document_from_url(
        fileModelId, fileUri, locale=fileLocale
    )
    ```

1. Locate the comment `Display invoice information to the user`. Following that, on news lines, enter the following code:

    **C#**

    ```csharp
    AnalyzeResult result = operation.Value;
    
    foreach (AnalyzedDocument invoice in result.Documents)
    {
        if (invoice.Fields.TryGetValue("VendorName", out DocumentField? vendorNameField))
        {
            if (vendorNameField.FieldType == DocumentFieldType.String)
            {
                string vendorName = vendorNameField.Value.AsString();
                Console.WriteLine($"Vendor Name: '{vendorName}', with confidence {vendorNameField.Confidence}.");
            }
        }
    ```

    **Python**

    ```python
    receipts = poller.result()
    
    for idx, receipt in enumerate(receipts.documents):
    
        vendor_name = receipt.fields.get("VendorName")
        if vendor_name:
            print(f"\nVendor Name: {vendor_name.value}, with confidence {vendor_name.confidence}.")
    ```

    > [!NOTE]
    > You've added code to display the vendor name. The starter project also includes code to display the *customer name* and *invoice total*.

1. Save the changes to the code file.

1. In the interactive terminal pane, ensure the folder context is the folder for your preferred language. Then enter the following command to run the application.

1. ***For C# only***, to build your project, enter this command:

    **C#**:

    ```powershell
    dotnet build
    ```

1. To run your code, enter this command:

    **C#**:

    ```powershell
    dotnet run
    ```

    **Python**:

    ```powershell
    python document-analysis.py
    ```

The program displays the vendor name, customer name, and invoice total with confidence levels. Compare the values it reports with the sample invoice you opened at the start of this section.