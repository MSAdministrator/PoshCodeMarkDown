---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 3047
Published Date: 2012-11-10t22
Archived Date: 2012-12-25t07
---

# get-mwsorder - 

## Description

get seller orders from amazon’s mws web services … with items optionally included.

## Comments



## Usage



## TODO



## class

``

## Code

`#
 #
 // --------------------------------------------------------------------------------------------------------------------
 // <copyright file="GetOrderCmdlet.cs" company="Huddled Masses">
 //   Copyright (c) 2011 Joel Bennett
 // </copyright>
 // <summary>
 //   Defines the Get-Order Cmdlet for Amazon Marketplace Orders WebService.
 // </summary>
 // --------------------------------------------------------------------------------------------------------------------
 
 namespace PoshOrders
 {
    using System;
    using System.Management.Automation;
 
    // I'm using Amazon's MarketplaceWebServiceOrders Apache-licensed web-service access code.
    using MarketplaceWebServiceOrders;
    using MarketplaceWebServiceOrders.Model;
 
    /// <summary>
    /// Get Amazon Marketplace Orders from the WebService
    /// </summary>
    [Cmdlet(VerbsCommon.Get, "Order")]
    public class GetOrderCmdlet : Cmdlet
    {
       // Access Key ID and Secret Access Key ID for the Amazon MWS
 
       /// <summary>The Amazon Access Key ID</summary>
       private const string AccessKeyId = "INSERT YOUR MWS ACCESS KEY HERE";
 
       /// <summary>The Amazon Secret Key</summary>
       private const string SecretAccessKey = "INSERT YOUR MWS SECRET ACCESS KEY HERE";
 
       /// <summary>The Application Name is sent to MWS as part of the HTTP User-Agent header</summary>
       private const string ApplicationName = "INSERT YOUR APPLICATION NAME HERE";
 
       /// <summary>The Application Version is sent to MWS as part of the HTTP User-Agent header</summary>
       private const string ApplicationVersion = "1.0";
 
       /// <summary>Default value for the service URL is for the USA</summary>
       private string serviceUrl = "https://mws.amazonservices.com/Orders/2011-01-01";
 
       /// <summary>The web service client</summary>
       private MarketplaceWebServiceOrdersClient marketplaceWebServiceOrdersClient;
 
       /// <summary>Default value for "Created After" is 1 day ago.</summary>
       private DateTime createdAfter = DateTime.MinValue;
 
       /// <summary>Default value for "Created Before" is infinitely in the future</summary>
       private DateTime createdBefore = DateTime.MaxValue;
 
       /// <summary>The Default order status is Unshipped or PartiallyShipped</summary>
       private OrderStatusEnum[] orderStatus = new[] { OrderStatusEnum.Unshipped, OrderStatusEnum.PartiallyShipped };
 
       /// <summary>
       /// Gets or sets the merchant id.
       /// </summary>
       /// <value>
       /// The merchant id.
       /// </value>
       [Parameter(Mandatory = true, Position = 0, ValueFromPipelineByPropertyName = true)]
       public string MerchantId
       {
          get; 
          set;
       }
 
       /// <summary>
       /// Gets or sets the marketplace id.
       /// </summary>
       /// <value>
       /// The marketplace id.
       /// </value>
       [Parameter(Mandatory = true, ValueFromPipelineByPropertyName = true)]
       public string[] Marketplace
       {
          get; 
          set;
       }
 
       /// <summary>
       /// Gets or sets the service URL.
       /// Defaults to the USA service URL.
       /// </summary>
       /// <value>
       /// The service URL.
       /// </value>
       [Parameter(Mandatory = false, ValueFromPipelineByPropertyName = true)]
       public string ServiceUrl
       {
          get
          {
             return this.serviceUrl;
          }
 
          set
          {
             this.serviceUrl = value;
          }
       }
 
       /// <summary>
       /// Gets or sets a date orders must prescede.
       /// </summary>
       /// <value>
       /// The date.
       /// </value>
       [Parameter(Mandatory = false, ValueFromPipelineByPropertyName = true)]
       public DateTime CreatedBefore
       {
          get
          {
             return this.createdBefore;
          }
 
          set
          {
             this.createdBefore = value;
          }
       }
 
       /// <summary>
       /// Gets or sets a date the orders must come aftercreated after.
       /// </summary>
       /// <value>
       /// The created after.
       /// </value>
       [Parameter(Mandatory = false, ValueFromPipelineByPropertyName = true)]
       public DateTime CreatedAfter
       {
          get
          {
             return this.createdAfter;
          }
 
          set
          {
             this.createdAfter = value;
          }
       }
 
       /// <summary>
       /// Gets or sets the order status.
       /// </summary>
       /// <value>
       /// The order status.
       /// </value>
       [Parameter(Mandatory = false, ValueFromPipelineByPropertyName = true)]
       public OrderStatusEnum[] OrderStatus
       {
          get
          {
             return orderStatus;
          }
 
          set
          {
             orderStatus = value;
          }
       }
 
       /// <summary>
       /// Gets or sets the buyer email.
       /// </summary>
       /// <value>
       /// The buyer email.
       /// </value>
       [Parameter(Mandatory = false, ValueFromPipelineByPropertyName = true)]
       public string BuyerEmail
       {
          get;
          set;
       }
 
       /// <summary>
       /// Gets or sets the seller order id.
       /// </summary>
       /// <value>
       /// The seller order id.
       /// </value>
       [Parameter(Mandatory = false, ValueFromPipelineByPropertyName = true)]
       public string SellerOrderId
       {
          get;
          set;
       }
 
       /// <summary>
       /// Gets or sets whether to include the Order Items
       /// </summary>
       /// <value>
       /// True to included the Order Items
       /// </value>
       [Parameter(Mandatory = false)]
       public SwitchParameter IncludeItems
       {
          get;
          set;
       }
 
       /// <summary>
       /// Implement the Begin step for PowerShell Cmdlets
       /// </summary>
       protected override void BeginProcessing()
       {
          base.BeginProcessing();
          var config = new MarketplaceWebServiceOrdersConfig
          {
             ServiceURL = "https://mws.amazonservices.com/Orders/2011-01-01"
          };
 
          this.marketplaceWebServiceOrdersClient = new MarketplaceWebServiceOrdersClient(
             ApplicationName, ApplicationVersion, AccessKeyId, SecretAccessKey, config);
       }
 
       /// <summary>
       /// Implement the Process step for PowerShell Cmdlets
       /// </summary>
       protected override void ProcessRecord()
       {
          base.ProcessRecord();
 
          /*
          // If no dates are specificied, start a day ago ...
          if (CreatedBefore == DateTime.MaxValue && CreatedAfter == DateTime.MinValue)
          {
             CreatedAfter = DateTime.UtcNow.AddDays(-1);
          }
          */
 
          // Amazon's policy doesn't allow calling with a CreatedBefore after two minutes before the request time
          if (CreatedBefore == DateTime.MaxValue)
          {
             CreatedBefore = DateTime.Now.AddMinutes(-2);
          }
 
          var orderRequest = new ListOrdersRequest
          {
             MarketplaceId = new MarketplaceIdList(),
             OrderStatus = new OrderStatusList(),
             BuyerEmail = this.BuyerEmail,
             SellerId = this.MerchantId,
             SellerOrderId = this.SellerOrderId,
             CreatedAfter = this.CreatedAfter,
             CreatedBefore = this.CreatedBefore
          };
 
          orderRequest.OrderStatus.Status.AddRange(OrderStatus);
          orderRequest.MarketplaceId.Id.AddRange(Marketplace);
 
          var response = marketplaceWebServiceOrdersClient.ListOrders(orderRequest);
 
          foreach (var order in response.ListOrdersResult.Orders.Order)
          {
             var output = new PSObject(order);
             if (IncludeItems.ToBool())
             {
                var itemRequest = new ListOrderItemsRequest
                {
                   AmazonOrderId = order.AmazonOrderId,
                   SellerId = this.MerchantId,
                };
 
                var items = marketplaceWebServiceOrdersClient.ListOrderItems(itemRequest);
                output.Properties.Add(new PSNoteProperty("Items", items.ListOrderItemsResult.OrderItems.OrderItem));
             }
 
             WriteObject(output);
          }
       }
    }
 }
`

