# DotNet SDK
Tamara .NET SDK is a wrapper for the Tamara API.
### Installation
There are two ways to install Tamara .Net SDK in this folder:
1. Reference your project to Tamara.Net.ClientSDK.dll
2. Install nuget package 'Tamara_Client_NetSDK.1.0.0.nupkg' with offline mode
### Usage
####Configurate in appsettings.json
Add a "tamaraPayment" field with baseUrl is Tamara payment's Url, apiToken & notificationPrivateKey are provided in our service.
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft": "Warning",
      "Microsoft.Hosting.Lifetime": "Information"
    }
  },
  "AllowedHosts": "*",
  "tamaraPayment": {
    "clientVersion": "1.0.0",
    "baseUrl": "https://api-sandbox.tamara.co",
    "requestTimeout": 10,
    "apiToken": "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiJ9.eyJhY2NvdW50SWQiOiI0NWQwMzAzOC1kM2I2LTQ1ODctYWY2Ny1hNDNlY2FlYjFiZDMiLCJ0eXBlIjoibWVyY2hhbnQiLCJzYWx0IjoiOTIwNDFjZDVlOTJlNDQ1MDg1ZTQ2NzgyZWFhYTY3NjkiLCJpYXQiOjE1OTIxMzM5NTEsImlzcyI6IlRhbWFyYSJ9.HASQ6UR1fabagwqkivmCqL4cFVDOw2cgBMRm5XPlJiNSfJ-gsgwqnPnEEn6-T7Sj4sU6Niee8pUPqP4_WsVQ6DojignLFg2cmrIS_dMIZyOXrZwMbhH6Y0fX5xt2yBVpEVjRbXVaEY4xgHNfMzLwz3mIqmJ-_xuwDDA-hGQk7xibbewDVdCDviSYRHSdzOPtIhy7dcx0CkyYWOncqpJ9YyePrrA1aqZeyWclxgAuZ6zYzsHM_o2e0zwZDqKi1spY11-s1ULSd1WmAWNwoKwy1C4jThJWlXl_E4-bPR_5CUcMHgy7Je9uN3Zfwuq7XODr7ShTnsEZ-xAQW7BxejqvIA",
    "notificationPrivateKey": "b28fbd1412c7e40ce7aa625c38763f71",
    "paths": {
      "getPaymentType": "/checkout/payment-types",
      "createCheckout": "/checkout",
      "authoriseOrder": "/orders/{orderId}/authorise",
      "cancelOrder": "/orders/{orderId}/cancel",
      "capture": "/payments/capture",
      "refund": "/payments/refund",
      "getOrderDetails": "/orders/{orderId}",
      "updateOrderReferenceId": "/orders/{orderId}/reference-id",
      "registerWebHook": "/webhooks",
      "retrieveWebhook": "/webhooks/{webhookId}",
      "removeWebhook": "/webhooks/{webhookId}",
      "updateWebhook": "/webhooks/{webhookId}"
    }
  }
}
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

