***The In-Person SDK for Windows is deprecated when integrated with the BBPOS Chipper 2X reader and ID Tech Augusta. The BBPOS Chipper 2X and ID Tech Augusta hardware is no longer available.***

# Overview

The In-Person Windows SDK enables your payment application to securely submit chip-card payments to Authorize.Net. Before you use this SDK, we recommend that you familiarize yourself with [Authorize.Net's C# SDK](https://github.com/AuthorizeNet/sample-code-csharp), which provides an interface to communicate with Authorize.Net.

This SDK is currently a certified solution with TSYS. To determine which processor you use, you can submit an API call to [getMerchantDetailsRequest](https://developer.authorize.net/api/reference/#transaction-reporting-get-merchant-details). The response contains a `processors` object.

For a list of frequently asked questions, see [our EMV FAQ page](https://support.authorize.net/s/article/Merchant-EMV-Chip-FAQs).

# Authentication

### **Transaction Key Authentication (Recommended)**

Authentication using a password and `mobileDeviceLoginRequest` is deprecated and will no longer be supported after **November 12, 2025**. You must update your integration to use a Transaction Key for authentication.

To get your credentials, log in to the Merchant Portal and navigate to **Account > Settings > API Credentials & Keys**. From there, you can obtain your **API Login ID** and generate a new **Transaction Key**.

Update your request's `merchantAuthentication` object to use the `transactionKey` as shown below.

```csharp
request.merchantAuthentication = new merchantAuthenticationType()
{
    name = "YOUR_API_LOGIN_ID",
    Item = "YOUR_TRANSACTION_KEY",
    ItemElementName = ItemChoiceType.transactionKey,
};
```

**Note: This Transaction Key authentication method should be used for all Authorize.Net API requests, such as `getUnsettledTransactionsRequest`, `getSettledBatchListRequest`, sending email receipts, and more.**

### **Password Authentication (Deprecated)**

The method of authenticating with a username, password, and `mobileDeviceId` to generate a `sessionToken` is now deprecated. Please migrate to Transaction Key authentication before November 12, 2025.

# Supported Encrypted Readers

Encrypted card readers supported by this SDK can be obtained from our [POS Portal](https://partner.posportal.com/authorizenet/auth/).

# Running the Sample App

The following steps will guide you on how to run the included sample app with the new Transaction Key authentication method.

### 1. Change the Startup URI in `App.xaml`

Open the `ANetEmvDesktopSdk.Sample` project. In the `App.xaml` file, change the `StartupUri` from `MainWindow.xaml` to `MainController.xaml`.

```xml
<Application x:Class="ANetEmvDesktopSdk.Sample.App"
             xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             xmlns:local="clr-namespace:ANetEmvDesktopSdk.Sample"
             StartupUri="/MainController.xaml"> <!-- This was /MainWindow.xaml before -->
    <Application.Resources>
         
    </Application.Resources>
</Application>
```

### 2. Initialize the SDK in `MainController.xaml.cs`

Open the `MainController.xaml.cs` file and add the following lines to the `public MainController()` constructor to initialize the SDK launcher.

```csharp
public MainController()
{
    InitializeComponent();
    this.WindowStartupLocation = WindowStartupLocation.CenterScreen;
    Application.Current.Exit += new ExitEventHandler(this.OnApplicationExit);
    
    // New added lines for initializing launcher
    Random random = new Random();
    this.amount.Text = (random.Next(1, 1000)).ToString();
    // Initialize with your desired environment, currency, terminal ID, and options
    this.launcher = new SdkLauncher(this.sdkEnvironment, "840", "", true, false);
    this.launcher.setMerchantInfo("YOUR_MERCHANT_NAME", "YOUR_MERCHANT_ID");
    this.launcher.enableLogging();
}
```
You can customize the `SdkLauncher` constructor with your specific settings:
*   `sdkEnvironment`: `AuthorizeNet.Environment.SANDBOX` or `AuthorizeNet.Environment.LIVE`.
*   `currencyCode`: The currency code for transactions (e.g., "840" for USD).
*   `terminalID`: Your assigned Terminal ID.
*   `skipSignature`: `true` to skip the signature screen.
*   `showReceipt`: `true` to display the SDK's receipt screen.

### 3. Update Merchant Authentication

In `MainController.xaml.cs`, find all method where `merchantAuthentication` is set (for example, in `getRequest()`) and update it to use your **API Login ID** and **Transaction Key** instead of a `sessionToken`. Here is the `getRequest()` method of `MainController.xaml.cs` updated:

```csharp
        private createTransactionRequest getRequest ()
        {
            Debug.Write("Session Token" + this.sessionToken);
            Random random = new Random();

            ApiOperationBase<ANetApiRequest, ANetApiResponse>.MerchantAuthentication = new merchantAuthenticationType()
            {
                name = "YOUR_API_LOGIN_ID",
                Item = "YOUR_TRANSACTION_KEY",
                ItemElementName = ItemChoiceType.transactionKey,
            };

            ApiOperationBase<ANetApiRequest, ANetApiResponse>.RunEnvironment = this.sdkEnvironment;

            transactionRequestType transaction = new transactionRequestType()
            {
                amount = Convert.ToDecimal(this.amount.Text, CultureInfo.InvariantCulture),
                transactionSettings = new settingType[] {
                },
                retail = new transRetailInfoType()
                {
                    deviceType = "7",
                    marketType = "2",
                },
                order = new orderType()
                {
                    description = "Windows SDK Order",
                    invoiceNumber = Convert.ToString(random.Next(1999999, 999999999))
                }
            };
            transaction.terminalNumber = this.terminalID;
            createTransactionRequest request = new createTransactionRequest()
            {
                clientId = "sdk-inperson-windows",
                transactionRequest = transaction
            };
            Debug.Write("Session Token" + this.sessionToken);
            request.merchantAuthentication = new merchantAuthenticationType()
            {
                name = "YOUR_API_LOGIN_ID",
                Item = "YOUR_TRANSACTION_KEY",
                ItemElementName = ItemChoiceType.transactionKey,
            };
            return request;
        }
```

### 4. Build and Run

Build and run the `ANetEmvDesktopSdk.Sample` project. The application will now bypass the old login screen and start directly, ready to process transactions.

# Integrating the SDK With Your Application

1.  Download the SDK.

2.  Copy the SDK folder into your project folder.
3.  Open the project in Visual Studio.
4.  Add a reference to `ANetEmvDesktop.dll`.
5.  Add the following references from NuGet Manager (Tools -> NuGet Package Manager -> Manage NuGet Packages for Solution):

```
i.  AuthorizeNet.dll
ii. BouncyCastle.Crypto.dll
iii.Microsoft.Bcl
iv. Microsft.Bcl.Async
v.  Microsoft.Bcl.Build
```

6.  If you use the IDTech_Augusta card reader, add the .dll references shown below from the `IDTechSdk` folder to your project. Then right-click your project and add the .dll references as existing items.

```
i.   Augusta_config.dll
ii.  Augusta_device.dll
iii. Augusta_emv.dll
iv.  Augusta_icc.dll
v.   Augusta_KSB_config.dll
vi.  Augusta_KB_device.dll
vii. Augusta_KB_msr.dll
viii.Augusta_KB_parse.dll
ix.  Augusta_msr.dll
x.   Augusta_parse.dll
```

8. Your application must implement the `SdkListener` interface and its methods to receive callbacks from the SDK during various processes.

```csharp
public interface SdkListener
{
    void transactionCompleted(createTransactionResponse response, bool isSuccess, string customerSignature, ErrorResponse errorResponse);
    void transactionStatus(TransactionStatus iTransactionStatus);
    void transactionCanceled();
    void hideCancelTransaction();
    void processCardProgress(TransactionStatus iProgress);
    void processCardCompletedWithStatus(bool iStatus);
    void requestSelectApplication(List<string> appList);
    void readerDeviceInfo(Dictionary<string, string> iDeviceInfo);

    // item1: Config update, item2: Firmware update
    void OTAUpdateRequired(Tuple<OTAUpdateResult, OTAUpdateResult> iCheckUpdateStatus, string iErrorMessage);
    void OTAUpdateProgress(double iPercentage, OTAUpdateType iOTAUpdateType);

    // item1: Config update, item2: Firmware update
    void OTAUpdateCompleted(Tuple<OTAUpdateResult, OTAUpdateResult> iUpdateStatus, string iErrorMessage);
}
```

# Initializing ANetEmvDesktopSdk

```csharp
launcher = new SdkLauncher(iEnvironment, iCurrencyCode, iTerminalID, iSkipSignature, iShowReceipt); 
```

*   `iEnvironment`: There are two environments - `SANDBOX` for testing your integration and `LIVE` for processing real transactions.
*   `iCurrencyCode`: Currency code of the country. For example, the USA currency code is 840.
*   `iTerminalID`: Terminal ID of the merchant terminal.
*   `iSkipSignature`: Set to `true` to skip the signature during checkout.
*   `iShowReceipt`: Boolean to enable or disable the receipt screen in the transaction flow.

# Set Reader Device Type

```csharp
// SDK supports two devices: AnywhereCommerce_Walker and IDTech_Augusta
// AnywhereCommerce_Walker is selected by default.
public void setReadername(ReaderName readerName) // Refer: SDKLauncher
```

# Set Terminal Mode

```csharp
// SDK allows Swipe or Insert_or_swipe. 
// Insert_or_swipe accepts chip-based transactions as well as Swipe/MSR transactions.
// Swipe accepts only MSR/Swipe transactions.
public void setTerminalMode(TerminalMode iTerminalCapability)

// Insert_or_swipe is selected by default.
// Refer to the SDKLauncher file and the sample app for more details.
```

# Set Reader Device Connection Type

```csharp
// Only AnywhereCommerce_Walker device supports two types of connection: USB and Bluetooth. 
// IDTech_Augusta only supports USB connection.
public void setReadername(ReaderName readerName) // Refer: SDKLauncher
```

# Set Up the Bluetooth Connection

```csharp
// Set the connection type by calling the following method:
public void setConnection(ConnectionMode iConnectionMode) // Refer: SDKLauncher

// Call the method below to discover the nearby devices and present the list to the user.
public void establishBTConnectionAndRetrieveNearByDevices(SdkListener iListener)

// On selection, call the following method to establish the connection with the device.
public void connectBTAtIndex(int iSelectedIndex) // Refer: SDKLauncher

// Implement the following methods of SDKListener: Refer SDKListener.
void BTPairedDevicesScanResult(List<BTDeviceInfo> iPairedDevicesList);  // Callback method which returns the near by devices
void BTConnected(BTDeviceInfo iDeviceInfo);  // Callback on Successful Bluetooth connection with the selected device
void BTConnectionFailed();  // Callback on failure of Bluetooth connection
```

Transaction Processing
========================

`ANetEmvDesktopSdk` can post transactions using two different options. In the first option, the SDK takes control and presents its own UI. In the second option, the SDK doesn’t show any UI; the SDK triggers the event about the transaction progress, and your application responds to these events.

Steps to post a transaction: 
-----------------------------

1.  Create a transaction object.

Update the `getRequest` method to use Transaction Key authentication.

```csharp
private createTransactionRequest getRequest()
{
    Random random = new Random();

    // Use Transaction Key for authentication
    ApiOperationBase<ANetApiRequest, ANetApiResponse>.MerchantAuthentication = new merchantAuthenticationType()
    {
        name = "YOUR_API_LOGIN_ID",
        Item = "YOUR_TRANSACTION_KEY",
        ItemElementName = ItemChoiceType.transactionKey,
    };

    ApiOperationBase<ANetApiRequest, ANetApiResponse>.RunEnvironment = this.sdkEnvironment;

    transactionRequestType transaction = new transactionRequestType()
    {
        amount = Convert.ToDecimal(this.amount.Text, CultureInfo.InvariantCulture),
        transactionSettings = new settingType[] { },
        retail = new transRetailInfoType()
        {
            deviceType = "7",
            marketType = "2",
        },
        order = new orderType()
        {
            description = "Windows SDK Order",
            invoiceNumber = Convert.ToString(random.Next(1999999, 999999999))
        }
    };
    transaction.terminalNumber = this.terminalID;
    createTransactionRequest request = new createTransactionRequest()
    {
        transactionRequest = transaction
    };

    // Use Transaction Key for authentication
    request.merchantAuthentication = new merchantAuthenticationType()
    {
        name = "YOUR_API_LOGIN_ID",
        Item = "YOUR_TRANSACTION_KEY",
        ItemElementName = ItemChoiceType.transactionKey,
    };
    return request;
}
```

2. Once the transaction object is populated, call one of the following
methods to post a transaction, passing the transaction object,
transaction type and `SDKListener`. Currently SDK only supports
GOODS, SERVICES, TRANSFER and PAYMENTS transaction type.

### Quick Chip Transaction With SDK-Provided UI

>     public void startQuickChipTransaction(createTransactionRequest iTransactionRequest, SDKTransactionType iTransactionType, SdkListener iListener);   

#### Listener/Callback Methods

>     void transactionCompleted(createTransactionResponse response, bool isSuccess, string customerSignature, ErrorResponse errorResponse);
>     void transactionCanceled();

### Quick Chip Transaction With No UI from the SDK

>     public void startQuickChipWithoutUI(createTransactionRequest iTransactionRequest, SDKTransactionType iTransactionType, SdkListener >    iListener);


#### Listener/Callback Methods

>     void transactionCompleted(createTransactionResponse response, bool isSuccess, string customerSignature, ErrorResponse errorResponse);
>     void transactionStatus(TransactionStatus iTransactionStatus);
>     void requestSelectApplication(List<string> appList);
>     void hideCancelTransaction();

### Cancel the Transaction
This method only applies when you use a Quick Chip transaction with no UI from the SDK. Your application can call this method to cancel the transaction. If the returned value is true then the transaction was canceled succesfully, else the SDK could not cancel the transaction.

>     public bool cancelTransaction()

### Process a Card
The SDK's Quick Chip funtionality enables your application to process the card data even before the final amount is ready. However, processing the card does not authorize or capture the transaction. It retrives the card data and stores it in in-flight mode inside the SDK. When your application is ready with the final amount, it must initiate a Quick Chip transaction to capture the processed card data. When your application calls the process card method, the following Quick Chip transaction charges the processed card data.

### Process Card with Predetermined Amount
>     public void processCardInBackground(SdkListener iListener, SDKTransactionType iTransactionType)

#### Listener/Callback Methods

>     void processCardProgress(TransactionStatus iProgress);
>     void processCardCompletedWithStatus(bool iStatus);
>     void requestSelectApplication(List<string> appList);

### Discard Processed Card Data
If the transaction is cancelled, your application must discard the processed card data.

>     public void discardProcessedCardData()

Firmware and Configuration Update
==================================

The SDK is capable of updating the reader device firmware and configuration. Your application can check if the reader device is up to
date or not by calling the following method of `SdkLauncher` class.

###   Check for Firmware or Configuration Updates

>     public void checkForAnywhereReaderDeviceUpdates(SdkListener iListener, bool isTestReader)

#### Listener/Callback Methods

>     Void OTAUpdateRequired(Tuple<TAUpdateResult, OTAUpdateResult> iCheckUpdateStatus, string iErrorMessage); 

## Update Required 

If an update is required, your application can call one of the following methods to update the configuration or firmware.

###   Start Firmware/Configuration Update With SDK-Provided UI

>     public void startOTAUpdate(SdkListener iListener, bool isTestReader)

#### Listener/Callback Methods

>     void OTAUpdateCompleted(Tuple<OTAUpdateResult, OTAUpdateResult> iUpdateStatus, string iErrorMessage);

### Start Firmware/Configuration Update With No UI From SDK

>     public void startOTAUpdateWithNoUI(SdkListener iListener, bool isTestReader)

#### Listener/Callback Methods

>     void OTAUpdateProgress(double iPercentage, OTAUpdateType
>     iOTAUpdateType);

>   //item1 Config update
>   //item2 Firmware update

>     void OTAUpdateCompleted(Tuple<OTAUpdateResult, OTAUpdateResult> iUpdateStatus, string iErrorMessage);


## Notes
For every SDK operation, the SDK has call back methods. The SDK sends notifications about the progress or completion of the operation in callback methods. Callbacks methods are listed under each operation. 


## Error  Codes

You can view these error messages at our [Reason Response Code Tool](http://developer.authorize.net/api/reference/responseCodes.html) by entering the specific Response Reason Code into the tool. There will be additional information and suggestions there.

Field Order	| Response Code | Response Reason Code | Text
--- | --- | --- | ---
3 | 2 | 355	| An error occurred while parsing the EMV data.
3 | 2 | 356	| EMV-based transactions are not currently supported for this processor and card type.
3 | 2 | 357	| Opaque Descriptor is required.
3 | 2 | 358	| EMV data is not supported with this transaction type.
3 | 2 | 359	| EMV data is not supported with this market type.
3 | 2 | 360	| An error occurred while decrypting the EMV data.
3 | 2 | 361	| The EMV version is invalid.
3 | 2 | 362	| x_emv_version is required.
