# HavenOnDemand library for Windows Universal 8.1. V2.0

Official ASP.NET client library to help with calling [Haven OnDemand APIs](http://havenondemand.com).

## What is Haven OnDemand?
Haven OnDemand is a set of over 70 APIs for handling all sorts of unstructured data. Here are just some of our APIs' capabilities:
* Speech to text
* OCR
* Text extraction
* Indexing documents
* Smart search
* Language identification
* Concept extraction
* Sentiment analysis
* Web crawlers
* Machine learning

For a full list of all the APIs and to try them out, check out https://www.havenondemand.com/developer/apis


## Overview
HODClient library for Windows Universal is a lightweight C# based API, which helps you easily access over 60 APIs from HPE HavenOnDemand platform.

The library contains 2 packages:

HODClient package for sending HTTP GET/POST requests to HavenOnDemand APIs.

HODResponseParser package for parsing JSON responses from HavenOnDemand APIs.

HODClient library requires the .NET 4.5.

----
## Integrate HODClient into a Windows/Windows Phone project
1. Right click on the project's References folder and select "Manage Nuget Packages...".
>![](/images/managenuget.jpg)

2. Select Browse and choose nuget.org for Package source then type in the search field "HavenOnDemand"
>![](/images/installhodnuget.jpg)

3. Select a package and click Install.

----
## Using HODClient package
```
using HOD.Client;
HODClient client = new HODClient("API_KEY", "v1");
```
where you replace "API_KEY" with your API key found [here](https://www.havenondemand.com/account/api-keys.html). `version` is an *optional* parameter which can be either `"v1"` or `"v2"`, but defaults to `"v1"` if not specified.

## Define and implement callback functions
You will need to implement callback functions to receive responses from Haven OnDemand server
```
client.requestCompletedWithContent += client_requestCompletedWithContent;
client.requestCompletedWithJobID += client_requestCompletedWithJobID;
client.onErrorOccurred += client_onErrorOccurred;
``` 

When you call the GetRequest() or PostRequest() with the "async = true" , the response will be returned to this callback function. The response is a JSON string containing the jobID.
```
private void HodClient_requestCompletedWithJobID(string response)
{
    // use the HODResponseParser to parse the response
    string jobID = parser.ParseJobID(response);
}
``` 

When you call the GetRequest() or PostRequest() with the "async = false", or call the GetJobResult() or GetJobStatus() functions, the response will be returned to this callback function. The response is a JSON string containing the actual result of the service.
```
private void HodClient_requestCompletedWithContent(string response)
{
    // use the HODResponseParser to parse the response
}
``` 

If there was an error occurred, the error message will be returned to this callback function.
```
private void HodClient_onErrorOccurred(string errorMessage)
{
    
}
```

If you want to change the API version without the need to recreate the instance of the HOD client.
```
SetAPIVersion(String version)
```
* `version` a string to specify a new API version as "v1" or "v2"

If you want to change the API_KEY without the need to recreate the instance of the HOD client.
```
SetAPIKey(String apiKey)
```
* `apiKey` a string to specify a new API_KEY

## Sending requests to the API - GET and POST
You can send requests to the API with either a GET or POST request, where POST requests are required for uploading files and recommended for larger size queries and GET requests are recommended for smaller size queries.

### Function GetRequest
```
void GetRequest(ref Dictionary<String, Object> Params, String hodApp, Boolean async = true, String version = "")
```
* `Params` is a dictionary object containing key/value pair parameters to be sent to a Haven OnDemand API, where the key is the name of a parameter of that API. 

>Note: For a value with its type is an array<>, the value must be defined in a List\<object\>. 
```
var entity_type = new List<object>();
entity_type.Add("people_eng");
entity_type.Add("places_eng");
var Params = new Dictionary<string, object>()
{
    {"url", "http://www.cnn.com" },
    {"entity_type", entity_type }
};
```

* `hodApp` a string to identify a Haven OnDemand API. E.g. "extractentities". Current supported apps are listed in the HODApps class.
* `async` [true | false]: specifies API call as Asynchronous or Synchronous. Default to true.
* `version` is a string to specify an API version. Can be omitted or an empty string.

*Example code:*
    Call the Entity Extraction API to find people and places from CNN and BBC website
```
String hodApp = HODApps.ENTITY_EXTRACTION;

var urls = new List<object>();
urls.Add("http://www.cnn.com");
urls.Add("http://www.bbc.com");

var entity_type = new List<object>();
entity_type.Add("people_eng");
entity_type.Add("places_eng");

var Params = new Dictionary<string, object>()
{
    {"url", urls },
    {"entity_type", entity_type }
};

hodClient.GetRequest(ref Params, hodApp, false);
```

### Function PostRequest
```
void PostRequestSync(ref Dictionary<String, Object> Params, String hodApp, Boolean async = true, String version="")
```
* `Params` is a dictionary object containing key/value pair parameters to be sent to a Haven OnDemand API, where the key is the name of a parameter of that API.

> Note:

> 1. In the case of the "file" parameter, the value must be a StorageFile object.
> 2. For a parameter with its type is an array<>, the parameter must be defined in a List\<object\>.
> E.g.:
```
var entity_type = new List<object>();
entity_type.Add("people_eng");
entity_type.Add("places_eng");
StorageFile file1 = await StorageFile.GetFileFromPathAsync("c:\doc1.txt");
StorageFile file2 = await StorageFile.GetFileFromPathAsync("c:\doc2.txt");
var files = new List<object>();
files.Add(file1);
files.Add(file2);
var Params = new Dictionary<string, object>()
{
    {"file", files },
    {"entity_type", entity_type }
};
```

* `hodApp` a string to identify a Haven OnDemand API. E.g. "ocrdocument". Current supported apps are listed in the HODApps class.
* `async` [true | false]: specifies API call as Asynchronous or Synchronous. Default to true.
* `version` is a string to specify an API version. Can be omitted or an empty string.

*Example code:*
    Call the OCR Document API to scan text from an image file
```
String hodApp = HODApps.OCR_DOCUMENT;
StorageFile file = await StorageFile.GetFileFromPathAsync("c:\image.jpg");
var Params =  new Dictionary<String,Object>
{
    {"file", file},
    {"mode", "document_photo"}
};
hodClient.PostRequest(ref Params, hodApp, true);
```

### Function GetRequestCombination

Sends a HTTP GET request for Haven OnDemand combination API.
```
void GetRequestCombination(ref Dictionary<String, Object> Params, String hodApp, Boolean async = true)
```
*Parameters:*
* `Params` is a Dictionary object containing key/value pair parameters to be sent to a combination API, where the keys are the parameters of that API
* `hodApp` is the name of the combination API you are calling.
* `async` [true | false] specifies API call as Asynchronous or Synchronous. Default to true.

### Function PostRequestCombination

Sends a HTTP POST request for Haven OnDemand combination API.
```
void PostRequestCombination(ref Dictionary<String, Object> Params, String hodApp, Boolean async = true)
```

*Parameters:*
* `Params` is a Dictionary object containing key/value pair parameters to be sent to a combination API, where the keys are the parameters of that API

> Note:

> 1. File upload is not yet supported in this version

* `hodApp` is the name of the combination API you are calling.
* `async` [true | false] specifies API call as Asynchronous or Synchronous. Default to true.

### Function GetJobResult

Sends a request to Haven OnDemand to retrieve content identified by the jobID
```
void GetJobResult(String jobID)
```

* `jobID` the jobID returned from a Haven OnDemand API upon an asynchronous call.

*Example code:*
    Parse a JSON string contained a jobID and call the function to get the actual content from Haven OnDemand server 
```
void hodClient_requestCompletedWithJobID(string response)
{
    JsonValue root;
    JsonObject jsonObject;
    if (JsonValue.TryParse(response, out root))
    {
        jsonObject = root.GetObject();
        string jobId = jsonObject.GetNamedString("jobID");
        hodClient.GetJobResult(jobId);
    }
}
```

### Function GetJobStatus

Sends a request to Haven OnDemand to retrieve status of a job identified by a job ID. If the job is completed, the response will be the result of that job. Otherwise, the response will be None and the current status of the job will be held in the error object. 
```
void GetJobStatus(String jobID)
```
* `jobID` the job ID returned from an Haven OnDemand API upon an asynchronous call.

*Example code:*
```
void hodClient_requestCompletedWithJobID(string response)
{
    JsonValue root;
    JsonObject jsonObject;
    if (JsonValue.TryParse(response, out root))
    {
        jsonObject = root.GetObject();
        string jobId = jsonObject.GetNamedString("jobID");
        hodClient.GetJobStatus(jobId);
    }
}

private void HodClient_requestCompletedWithContent(string response)
{

}
``` 

## Using HODResponseParser package
```
using HOD.Response.Parser;
HODResponseParser parser = new HODResponseParser();
```

### Function ParseJobID
```
string ParseJobID(string response)
```
* `response` a json string returned from an asynchronous API call.

*Return value:*
* The jobID or an empty string if not found.

*Example code:*

```
void hodClient_requestCompletedWithJobID(string response)
{
    string jobID = parser.ParseJobID(response);
    if (jobID != "")
        hodClient.GetJobResult(jobID);
}
```

## Parse Haven OnDemand APIs' response

Parses a json string and returns a class object.

*Example code:*

```
// 
void client_requestCompletedWithContent(string response)
{
    OCRDocumentResponse resp = parser.ParseOCRDocumentResponse(ref response);
    if (resp != null)
    {
        string text = "";
        foreach (OCRDocumentResponse.TextBlock obj in resp.text_block)
        {
            text += String.Format("Recognized text: {0}\n", obj.text);
            text += String.Format("Top/Left corner: {0}/{1}\n", obj.left, obj.top);
            text += String.Format("Width/Height: {0}/{1}\n", obj.width, obj.height);
        }
	Response.Write(text);
    }
    else
    {
        var errors = parser.GetLastError();
        foreach (HODErrorObject err in errors)
        {
            if (err.error == HODErrorCode.QUEUED)
            {
                // Task is in queue. Let's wait for a few second then call GetJobStatus() again
                await client.GetJobStatus(err.jobID);
                break;
            }
            else if (err.error == HODErrorCode.IN_PROGRESS)
            {
                // Task is In Progress. Let's wait for some time then call GetJobStatus() again
                await client.GetJobStatus(err.jobID);
                break;
            }
            else // It is an error. Let's print out the error code, reason and detail
            {
                var text += err.error.ToString() + "<br/>";
                text += err.reason + "<br/>";
                text += err.detail + "<br/>";
		Response.Write(text);
            }
        }
    }
}
```

**Function ParseCustomResponse**
```
object ParseCustomResponse<T>(jsonStr)
```

* `<T>`: a custom class object.
* `jsonStr` a json string returned from Haven OnDemand APIs.

*Example code:*
```
// define a custom class for Query Text Index API response
public class QueryIndexResponse
{
    public List<Documents> documents;
    public int totalhits { get; set; }
    public class Documents
    {
        public string reference { get; set; } 
        public string index { get; set; }
        public double weight { get; set; } 
        public List<string> from { get; set; } 
        public List<string> to { get; set; } 
        public List<string> sent { get; set; } 
        public List<string> subject { get; set; } 
        public List<string> attachment { get; set; }
        public List<string> hasattachments { get; set; }
        public List<string> content_type { get; set; }
        public string content { get; set; } 
    }
}
void client_requestCompletedWithContent(string response)
{
    QueryIndexResponse resp = (QueryIndexResponse) parser.ParseCustomResponse<QueryIndexResponse>(ref response);
    if (resp != null)
    {
        foreach (QueryIndexResponse.Documents doc in resp.documents)
        {
	    // walk thru documents array
            var reference = doc.reference;
            var index = doc.index;
            var weight = doc.weight;
            if (doc.from != null)
            {
                var from = doc.from[0];
            }
            if (doc.to != null)
                var to = doc.to[0];
            
            // parse any other values
        }
    }
    else
    {
        var errors = parser.GetLastError();
        foreach (HODErrorObject err in errors)
        {
            if (err.error == HODErrorCode.QUEUED)
            {
                // Task is in queue. Let's wait for a few second then call GetJobStatus() again
                client.GetJobStatus(err.jobID);
                break;
            }
            else if (err.error == HODErrorCode.IN_PROGRESS)
            {
                // Task is In Progress. Let's wait for some time then call GetJobStatus() again
                client.GetJobStatus(err.jobID);
                break;
            }
            else // It is an error. Let's print out the error code, reason and detail
            {
                var text += err.error.ToString() + "<br/>";
                text += err.reason + "<br/>";
                text += err.detail + "<br/>";
		Response.Write(text);
            }
        }
    }
}
```
---
## Demo code 1: 

**Call the Entity Extraction API to extract people and places from cnn.com website with a synchronous GET request**
```
using HOD.Client;
using HOD.Response.Parser;

namespace HODClientDemo
{    
    public sealed partial class MainPage : Page
    {
        HODClient hodClient = new HODClient("your-apikey");
        HODResponseParser parser = new HODResponseParser();

        public MainPage()
        {
            this.InitializeComponent();
                
            hodClient.requestCompletedWithContent += HodClient_requestCompletedWithContent;
            hodClient.requestCompletedWithJobID += HodClient_requestCompletedWithJobID;
            hodClient.onErrorOccurred += HodClient_onErrorOccurred;

            useHODClient();
        }

        private void useHODClient()
        {
            String hodApp = HODApps.ENTITY_EXTRACTION;
                
            var entity_type = new List<object>();
            entity_type.Add("people_eng");
            entity_type.Add("places_eng");

            var Params = new Dictionary<string, object>()
            {
                { "url", "http://www.cnn.com" },
                { "entity_type", entity_type },
                { "unique_entities", "true" }
            };

            hodClient.GetRequest(ref Params, hodApp, false);
        }

        // implement callback functions

        private void HodClient_requestCompletedWithContent(string response)
        {
            var resp = parser.ParseEntityExtractionResponse(ref response);
            if (resp != null)
            {
                String people = "";
                String places = "";
                foreach (var entity in resp.entities)
                {
                    if (entity.type == "people_eng")
                    {
                        people += entity.original_text + System.Environment.NewLine;
                        // parse any other interested information about a people
                    }
                    else if (entity.type == "places_eng")
                    {
                        places += entity.original_text + System.Environment.NewLine;
                        // parse any other interested information about a place
                    }
                }
            } 
	    else
            {
                var errors = parser.GetLastError();
                foreach (HODErrorObject err in errors)
                {
                    if (err.error == HODErrorCode.QUEUED)
                    {
                        // Task is in queue. Let's wait for a few second then call GetJobStatus() again
                        hodClient.GetJobStatus(err.jobID);
                        break;
                    }
                    else if (err.error == HODErrorCode.IN_PROGRESS)
                    {
                        // Task is In Progress. Let's wait for some time then call GetJobStatus() gain
                        hodClient.GetJobStatus(err.jobID);
                        break;
                    }
                    else 
                    {
                        // It is an error. Check error info and handle error accordingly
                    }
	        }
            }
        }
        private void HodClient_onErrorOccurred(string errorMessage)
        {
	    // handle error if any
        }
    }
}
```

## Demo code 2:
 
**Call the OCR Document API to recognize text from an image with an asynchronous POST request**
```
using HOD.Client;
using HOD.Response.Parser;

namespace HODClientDemo
{
    public sealed partial class MainPage : Page
    {
        HODClient iodClient = new HODClient("your-apikey");
	HODResponseParser parser = new HODResponseParser();
        StorageFile imageFile;
        public MainPage()
        {
            this.InitializeComponent();
                
            hodClient.requestCompletedWithContent += HodClient_requestCompletedWithContent;
            hodClient.requestCompletedWithJobID += HodClient_requestCompletedWithJobID;
            hodClient.onErrorOccurred += HodClient_onErrorOccurred;
        }

        private void useHODClient()
        {
            String hodApp = hODApps.OCR_DOCUMENT;
            StorageFile imgFile = StorageFile.GetFileFromPathAsync("path/and/filename");
            var Params = new Dictionary<string, object>
            {
                { "file", imageFile },
                { "mode", "document_photo" }
            };

            hodClient.PostRequest(ref Params, hodApp, true);
        }

        private async void LoadFileButton_Clicked(object sender, RoutedEventArgs e)
        {
            FileOpenPicker filePicker = new FileOpenPicker();
            filePicker.SuggestedStartLocation = PickerLocationId.PicturesLibrary;
            filePicker.FileTypeFilter.Add(".jpg");
            filePicker.FileTypeFilter.Add(".png");
            filePicker.ViewMode = PickerViewMode.Thumbnail;

            imageFile = await filePicker.PickSingleFileAsync();

            if (imageFile != null)
            {
                useHODClient();
            }
            else
            {
                // Cancel picking file
            }
            
        }            
        
	// implement callback functions
            
        /**************************************************************************************
        * An async request will result in a response with a jobID. We parse the response to get
        * the jobID and send a request for the actual content identified by the jobID.
        **************************************************************************************/ 
        private void HodClient_requestCompletedWithJobID(string response)
        {
            string jobID = parser.ParseJobID(response);
            if (jobID != "")
                client.GetJobResult(jobID);
        }

        private void HodClient_requestCompletedWithContent(string response)
        {
            OCRDocumentResponse resp = parser.ParseOCRDocumentResponse(ref response);
	    if (resp != null)
	    {
                var text = "";
                foreach (OCRDocumentResponse.TextBlock obj in resp.text_block)
                {
                    text += String.Format("Recognized text: {0}\n", obj.text);
                    text += String.Format("Top/Left corner: {0}/{1}\n", obj.left, obj.top);
                    text += String.Format("Width/Height: {0}/{1}\n", obj.width, obj.height);
                }
            }
            else
            {
                var errors = parser.GetLastError();
                foreach (HODErrorObject err in errors)
                {
                    if (err.error == HODErrorCode.QUEUED)
                    {
                        // Task is in queue. Let's wait for a few second then call GetJobStatus() again
                        hodClient.GetJobStatus(err.jobID);
                        break;
                    }
                    else if (err.error == HODErrorCode.IN_PROGRESS)
                    {
                        // Task is In Progress. Let's wait for some time then call GetJobStatus() gain
                        hodClient.GetJobStatus(err.jobID);
                        break;
                    }
                    else 
                    {
                        // It is an error. Check error info and handle error accordingly
                    }
	        }
            }
        }

        private void HodClient_onErrorOccurred(string errorMessage)
        {
            // handle error if any
        }
    }
}
```

## License
Licensed under the MIT License.