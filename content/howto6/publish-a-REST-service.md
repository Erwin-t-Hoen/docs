---
title: "Publish a REST Service"
#category: "Enter the category under which the document should be published (for example, "Mobile"); if there is a parent, remove this category line"
#parent: "Enter the parent document filename of this document if necessary (for example, "design-the-architecture"); if there is a category, remove this parent line"
#menu_order: Enter the position of the document in the category or under the parent; number by 10 (for first), 20, 30, etc. for easy ordering of other documents in the future if necessary; don't add brackets or quotation marks; if no number is added, the system will add an extremely high number to order the documents, which means that if you only want a document to appear at the top, you only have to add "10" to that specific document, you don't have to order all the other documents in the category/under the parent
#description: "Set a description with a maximum of 140 characters; this should describe what the goal of the document is, and it can be different from the document introduction; this is optional, and it can be removed"
#tags: [Add a maximum of 5-7 tags/keywords; keep them focused on the most important topics of the document; each tag should have quotation marks and be separated by a comma, for example: "Samba", "MxCloud", "cloud", "share"; the tags should be enclosed with brackets]
---

## 1 Introduction

Mendix allows you to publish REST web services natively from the modeler. This how-to will show you how to publish a REST service in an example project. This example will demonstrate the GET operation for a published REST service.

**This how-to will teach you how to do the following:**

* Create a published REST service and return the reults in JSON or XML

## 2 Prerequisites


Before starting this how-to, make sure you have completed the following prerequisites:

* Install Modeler version 7.11 or higher

## 3 Setting up the example project

In this section you will create the example project to be used in the next sections for publishing your REST service.

1. Create a new project in the modeler
2. Rename the MyFirstModule module to RESTExample
3. Open the Domain model of the RESTExample module
4. Create entities an associations to match the example below 

![](attachments/publish-a-REST-service/domainmodel.png)


Create the following pages to enter some order data:
1. In the RESTExample module create an overview page for the Order entity
2. Create a NewEdit page for the Orders, add a datagrid to the Order_NewEdit page that displays the OrderItems over association

Your page now looks like this:
![](attachments/publish-a-REST-service/order_NewEdit_Page.png)

Add the overview page to your project navigation and run the application. Create a couple of orders and orderlines by filling out the appropriate fields.
![](attachments/{sub-folder with same name as doc file}/{image filename}.png)


## 4 Publish the Service

To be able to use the data from your model in the REST service you need to create a Message Definition.

### 4.1 Create the mapping

  1. In the project Explorer right click on the RESTExample module and navigate to Add > Mappings > Message Definitions, like below:
  ![](attachments/publish-a-REST-service/message_definition.png)
  2. In the Add Message Definition dialog that is now show enter the name for this definition: MD_Orders
  3. The Message Definition is now opened and you need to select the Entity you will be using for the MD_Orders Message Definition
   ![](attachments/publish-a-REST-service/MD_AddEntity.png)
   First press the Add button and in the dialog press the select button and choose the Order Entity from the list
  4. After selecting the Order Entity the Structure part of the dialog is filled with only the Order object selected.
  5. Select the OrderID and the Customer attributes
  6. Expand the association OrderItem_Order and select the attributes Product and Quantity.

   ![](attachments/publish-a-REST-service/MD_SelectedAttributes.png)

   7. Press Ok to close the dialog
   8. Close the message definition and make sure to save the definition if asked!

### 4.2 Now for the REST Service

   1.  In the project Explorer right click on the RESTExample module and navigate to Add > Published Services > Published REST Service, like below:
    ![](attachments/publish-a-REST-service/AddRestService.png)
   2. In the rename dialog name your REST service PRS_OrderService and press OK. The REST service is now opened
   3. Now add a new resource to your service by pressing the Add button and name the resource GetOrderByID, like below
    ![](attachments/publish-a-REST-service/AddRestResource.png)
    Press OK to close the dialog
   4. Now add an operation to your resourceby pressing the add button in to operations for resource section, like below:
    ![](attachments/publish-a-REST-service/AddOperation.png)
   5. Add {OrderID} in the Operation path field, make sure to include the curly braces. This will allow the REST service to be invoked with the OrderID in the URL as is shown in the Example location field of the dialog
   6. In the same dialog use the select button next to the Microflow field. As you do not have a microflow for this operation select the new button in the following dialog and make sure to add the new microflow in the RESTExample module with the name: PRS_GetGetOrderByID
   ![](attachments/publish-a-REST-service/AddOperationMicroflow.png)
   7. Press the Show button to Edit the newly created Microflow, close the Operation dialog with the OK button.
   8. The microflow has 2 parameters: httpRequest and OrderID and returns an object of type httpResponse.
   9. Add a first action to the microflow to convert the OrderID variable (a String) to an Integer variable. The is needed to be able to search for the OrderID (an Autonumber)
   ![](attachments/publish-a-REST-service/ConvertOrderID.png)
   10. Add a second activity to the microflow to retrieve the Order based on the OrderID. This activity is a retrieve action from the database that returns 1 Order.
   ![](attachments/publish-a-REST-service/RetrieveOrder.png)
   11. From the project explorer right click the RESTExample module and navigate to Add > Mappings > Export Mapping to add a new mapping named EM_ExportOrder
   ![](attachments/publish-a-REST-service/AddExportMapping.png)
   12. In the Select schema elements for export mapping dialog select the Message Definition option and select the MD_Orders mapping created earlier via the select button.
   ![](attachments/publish-a-REST-service/SelectSchemaForExport.png)
   Make sure to select all the attributes a shown above and press OK
   13. In the Export Mapping that is now shown map the objects to the same objects from the domain model (either double click or drag and drop from the connector panel). Make sure to map the attributes with the same names. Your mapping should be the same as below:
   ![](attachments/publish-a-REST-service/ExportMappingResult.png)
   14. Now go back to the microflow PRS_GetGetOrderByID and add a third activity this is a Export wit Mapping activity.
   ![](attachments/publish-a-REST-service/MFExportWithMapping.png)
   In the Mapping field select the mapping created in step 11. For the parameter field seelct the Order object retrieved with the database retrieve action in the microflow. In this how-to we'll use JSON as the result and we'll store the output in a String variable named Order_JSON
   15. Add a fourth activity creating an object of type httpResponse as below:
   ![](attachments/publish-a-REST-service/httpResponse.png)
   The StatusCode will return an OK as a 200 message. The content of the message is mapped to the exported json from step 14. And add the HttpVersion that you will be using, in this case HTTP/1.1
   16. Add the next action to the microflow to add a header to the response.
   ![](attachments/publish-a-REST-service/httpResponseHeader.png)
   Set the member Key to 'Content-Type' and the Value to 'application/json' (or 'application/xml' if you're response contains XML rather than JSON). Set the System.HttpHeaders association to your HTTP response.
   17. Now open the end activity in your microflow and select the $NewHttpResponse as the return value. 
   Now you have no Errors and a microflow that is similar to the one below:
   ![](attachments/publish-a-REST-service/CompleteMFNoErrorHandling.png)

### 4.3 Let's try it out
   Restart your project and open it in the browser via the following URL: http://localhost:8080/rest-doc/
   This will show you the page with the documentation of all your published REST services:
   ![](attachments/publish-a-REST-service/RESTTestOverview.png)
   
   Click on the PRS_OrderService link to view the details

   ![](attachments/publish-a-REST-service/RESTTestDetails.png)

   Press the GET button followed by the 'Try it out' button.
   Fill in a OrderID and press the Execute button.
   ![](attachments/publish-a-REST-service/RESTTestExceute.png)

   This will execute the request and return the result in the Response body:
   ![](attachments/publish-a-REST-service/RESTTestResponse.png)

Cool you have publish your first REST service from Mendix.

## 5 Error Handling

In this new service no error handling has been implemented this needs to be done in order to publish a robust service. For example if you now execute your service as under 4.3 and enter some string in the OrderID parameter or leave it out you will see a 500 or a 404 generic error.

### 5.1 Add some error handling

   1. Open the microflow PRS_GetGetOrderByID and by right clicking the first activity select the option to set the Error Handling to Custom with roll back.
   2. From the first activity add an activity that creates a new httpResponse object and name this httpErrorResponse, with the atrributes mapped as in the next example:
   ![](attachments/publish-a-REST-service/ParsingErrorResponse.png)
   For the content the value is a json string:

   `'{"Error": "The OrderID can only be an integer"}'`
   Be sure to set the new activity as the custom error handler!
   3. Below this actiity add a create object activity that creates a new httpHeader object
   ![](attachments/publish-a-REST-service/ParsingErrorResponseHeader.png)
   Make sure to associate the header to the NewHttpErrorResponse
   4. Add a new endpoint for the microflow and set the NewHttpErrorResponse as the return value. And your microflow now looks like this:
   ![](attachments/publish-a-REST-service/ParsingErrorMicroflow.png)

   Test your error handler like under 4.3 and enter some characters in the OrderID parameter and you see that the response of the request is:
   ![](attachments/publish-a-REST-service/ParsingErrorRESTResult.png)

### 5.2 Additional Error handling

Now that you covered the error handling of the parameter parsing, it's time to handle empty responses. This is not really an error, but some indication of what happened when nothing is returned is always is good idea.
In the next steps you'll add the error handling for those situations where when the OrderID parameter is filled but no result is found.

   1. After the retrieve from database activity add an exclusive split activity with the following statement:

   `$Order != empty`
   
   The true exit is connected to the export to JSON activity and on the false exit add a new create object activity creating a new httpErrorNotFoundResponse and a httpErrorNotFoundHeader
   ![](attachments/publish-a-REST-service/OrderNotFoundResponse.png)

   The content is filled with the following string:

   `'{"Error": "No Order available for ID:'+$OrderID+'"}'`

   ![](attachments/publish-a-REST-service/OrderNotFoundHeader.png)

  2. Make sure the end activity returns the new httpResponse
  3. The microlfow now looks like:
    ![](attachments/publish-a-REST-service/CompleteMFWithErrorHandling.png)
  4. Test your new error responses in the same way as mentioned under 4.3

# 6 Related Content
For additional information on creating published REST services in Mendix including GET, POST and DELETE operations watch the video on youtube from TimeSeries: https://youtu.be/zt1XBnK2LCM

{Do not enter anything here, this will be generated by Mendix}
