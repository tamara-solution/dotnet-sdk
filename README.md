# DotNet SDK
Tamara .NET SDK is a wrapper for the Tamara API.
### Installation
There are two ways to install Tamara .Net SDK in this folder:
1. Reference your project to Tamara.Net.ClientSDK.dll
2. Install nuget package 'Tamara_Client_NetSDK.1.0.0.nupkg' with offline mode
### Usage
There are two ways:
##### 1. Use factory
- Create a ApiConfiguration instance
- Create a Logger instance
```
    var apiConfiguration = new ApiConfiguration();
    var builder = new ConfigurationBuilder()
        .SetBasePath(Path.Combine(AppContext.BaseDirectory))
        .AddJsonFile("appsettings.json", optional: true);
    IConfiguration configuation = builder.Build();
    configuation.GetSection("tamaraPayment").Bind(apiConfiguration);

    var loggerFactory = LoggerFactory.Create(builder =>
    {
        builder.AddLog4Net(new Log4NetProviderOptions()
        {
            Name = "TamaraApiClient",
            Log4NetConfigFileName = "App_Data/log4net.xml"
        });
    });
    var logger = loggerFactory.CreateLogger<ClientTest>();
   _apiClient = CheckoutServiceFactory.CreateClient(apiConfiguration, logger);
```
##### 2. Use dependency injection
ApiConfiguration & Logger were created as the same way above.
```
services.AddScoped<ITamaraApiClient, TamaraApiClient>(provider => new TamaraApiClient(apiConfiguration, logger));
```
#### Notification Service
##### 1. Register notification service
```
services.AddScoped<ITammaraNotificationService, TamaraNotificationService>(provider => new TamaraNotificationService(apiConfiguration.NotificationPrivateKey, logger));
```
##### 2. Create a controller to receive notification from Tamara
```
        [HttpPost]
        public async Task<IActionResult> ReceiveAuthoriseNotification()
        {
            var request = HttpContext.Request;
            var result =  await _tammaraNotificationService.ProcessAuthoriseNotification(request);

            return Json(result);
        }
        [HttpPost]
        public async Task<IActionResult> ReceiveWebhookNotification()
        {
            var request = HttpContext.Request;
            var result = await _tammaraNotificationService.ProcessWebhook(request);

            return Json(result);
        }
```
##### 3. Test:
POST: https://localhost:2600/Notification/ReceiveNotification
Header: 
```
Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE2MDEzMDYwOTcsImlhdCI6MTYwMTMwNDI5NywiaXNzIjoiVGFtYXJhIn0.JuoMMVuStl5RLKxfNDRH7nxjFaQBp4YQr-FH9DDR9zk
```
Body:
```
{
  "order_id": "16280ad8-cde0-4d27-b06d-e6658fe53382",
  "order_reference_id": "50548981",
  "order_status": "approved",
  "data": []
}
```

