# Using Watson Personality Insights Service with Watson Explorer

The [Watson Personality Insights Service](http://www.ibm.com/smarterplanet/us/en/ibmwatson/developercloud/personality-insights.html) uses linguistic analytics to infer cognitive and social characteristics, including Big Five, Values, and Needs, based on text information created by individuals or groups.  These characteristics can be used to generate actionable insights, for example in the context of [psychographic segmentation](http://en.wikipedia.org/wiki/Psychographic). The current Watson Personality Insights Service is trained using social media status updates such as from Twitter or Facebook, though other data sets that are representative of a user might be used.  The generated insights might be used as is or combined with other data to generate insights into possible purchasing behaviors, likes, dislikes, and other relevant customer interactions.  It is important to note that the generated portrait is only a representation of a person based on information available from a specific context (e.g. Twitter).  While there may be correlations between a customer portrait and certain observable behaviors, a generated model should never be used to evaluate psychological wellbeing of a person or to provide psychoanalysis services.

In the context of Watson Explorer, psychographic segmentation and customer portraits might prove very useful when combined with a 360 degree view of a customer or other (human) entities. Watson Explorer Application Builder already allows you to view relevant information from disparate data sources from a single web page. Adding a portrait generated by Watson Personality Insights layers _insights_ on top of the data you already have.

The goal of this example is to demonstrate how to get started with an integration between Watson Explorer and the Watson Personality Insights service available on IBM Bluemix.  By the end of the example you will have added two widgets to a Watson Explorer Application Builder application.  One widget creates and displays a portrait based on information indexed in Engine provided by an entity. A second widget demonstrates how users can generate a user portrait in real time based on a Twitter handle that is provided in the context of an entity page (simulating an in-progress user/customer interaction).



## Prerequisites
Please see the [Introduction](/intro-to-bluemix-for-app-builder.md) for an overview of the integration architecture, and the tools and libraries that need to be installed to create Java-based applications in Bluemix.

- An [IBM Bluemix](https://ace.ng.bluemix.net/) account
- [Watson Explorer](http://www-01.ibm.com/support/knowledgecenter/SS8NLW_11.0.0/com.ibm.swg.im.infosphere.dataexpl.install.doc/c_install_wrapper.html) - Installed, configured, and running
- A basic search application configured in Application Builder. For the purposes of this example, the [example-metadata based tutorial application](http://www-01.ibm.com/support/knowledgecenter/SS8NLW_11.0.0/com.ibm.swg.im.infosphere.dataexpl.appbuilder.doc/c_de-ab-devapp-tutorial.html) is sufficient.
- (Optional) [Prerequisites for creating a Bluemix Ruby based application](/intro-to-bluemix-for-app-builder.md#ruby-sinatra-web-based-applications). This prerequisite is required if you would like to develop Ruby applications on your local machine.
- (Optional) A Twitter account with API access.  This will be used to fetch status updates given a user's Twitter handle in one of the example widgets.

If you are using a version of Watson Explorer less than 11.0 you will also need to install and configure the [WEX App Builder Sample Proxy](https://github.com/IBM-Watson/wex-appbuilder-sample-proxy)

## What's Included in this Tutorial

This tutorial will walk through the creation and deployment of three components.

1. A basic Bluemix application exposing the Watson Personality Insights Service as a web service.
2. A custom Application Builder widget that sends text from an Application Builder entity to your Bluemix Personality Insights web service.
3. A custom Application Builder widget that takes a Twitter handle as input and creates a model based on a user's Twitter status updates at run time.


## Step-by-Step Tutorial

This section outlines the steps required to create a basic Watson Personality Insights widget in Application Builder and connect it with a custom Bluemix web service.

   
### Configuring and Deploying the BlueMix Custom Watson Personality Insights Web Service

The example Bluemix application uses a `manifest.yml` file to specify the application name, services bindings, and basic application settings.  Using a manifest simplifies distribution and deployment of CloudFoundry applications (for example, Bluemix).  Since the example application is written in Ruby, the code can be deployed to Bluemix as is. Ruby development tools are only required if you would like to develop and test the application locally.

If you have not already, sign in to Bluemix.

```
$> cf api api.ng.bluemix.net
cf login
```


Once you are signed in, you will need to create the Watson Personality Insights service that the example application will bind to.  In this example, we're calling the service `wex-personality-insights`. This name is already set in the `manifest.yml`.  Since services might be used by multiple applications, this name isn't ideal (a more descriptive name can improve maintainability), but it's perfectly suitable for this example.

```
$> cf create-service "personality_insights" "standard" wex-personality-insights
```

Note that the Watson Personality Insights service is a for-fee service, meaning you may be charged based on the service policies and rates.  You can review these policies in the Bluemix dashboard and documentation.

Next, deploy the application to your space in the Watson Developer Cloud.  If this is the first time deploying, the application will be created for you.  Subsequent pushes to Bluemix will overwrite the previous instances you deployed.

```
$> cf push --random-route
```


Once the application has started, you should now be able to run a test using the simple application test runner included in the application.  You can see the route that was created for your application with `cf routes`.  The running application URL can be determined by combining the host and domain from the routes listing.  The route is also available from your application dashboard in Bluemix.


#### Configuring the BlueMix Application for Twitter API Access

The Twitter enabled widget requires a valid [Twitter API](https://dev.twitter.com/) token for use.  Follow these steps to create a Twitter API application and update the example application code to use your Twitter API credentials.  This step is optional if you do not want to access Twitter.

1. Sign in to [https://dev.twitter.com/](https://dev.twitter.com/) using your Twitter username and password.
2. Once you are logged in you can view "My applications" from [https://apps.twitter.com/](https://apps.twitter.com/) directly.
3. Create a new App.
4. Fill out the form and agree to Twitter's Terms of Service.  A Callback URL is not required for this sample application.
5. The next page will show a summary of your Twitter API application.  Click on the "Keys and Access Tokens" tab.
6. The `Consumer Key (API Key)` and `Consumer Secret (API Secret)` are what you'll need to connect your Watson Bluemix application to Twitter.  Change lines 17 and 18 of [application_controller.rb](/personality_insights/BlueMix/controllers/application_controller.rb) to reflect your `API key` and `API secret`. It is not necessary to generate a Twitter access token for the integration. 


```ruby
set :twitter_api_key, "YOUR API KEY GOES HERE"
set :twitter_api_secret, "YOUR API SECRET GOES HERE"
```


At this point your Bluemix application should be able to use the Twitter API to directly download status updates with a valid Twitter handle.  Push your updated application up to Bluemix using `cf push` and test the newly deployed application.

Please note that the Twitter API has limits on the number of requests that can be made per hour.  The example code will fail gracefully when these limits and other common API errors occur, but the responsibility for managing your Twitter API and usage is up to you. For more information on the Twitter API see [https://dev.twitter.com/](https://dev.twitter.com/). The example Application Builder widget uses the excellent [Twitter Gem](http://sferik.github.io/twitter/).


### Configuring Watson Explorer Engine

This example requires that `example-metadata` is crawled and configured for use in Application Builder.  The Application Builder Tutorial can be found in the [Watson Explorer documentation.](http://www-01.ibm.com/support/knowledgecenter/SS8NLW_11.0.0/com.ibm.swg.im.infosphere.dataexpl.appbuilder.doc/c_de-ab-devapp-tutorial.html) 

### Configuring Watson Explorer Application Builder

This example includes two widgets.  Both custom widgets require the use of the App Builder Endpoints.  If you are using a version of Watson Explorer < 11.0, you will need to install and configure the [WEX App Builder Sample Proxy](https://github.com/IBM-Watson/wex-appbuilder-sample-proxy) instead.

#### Endpoints Configuration (WEX 11+)

Two sample endpoints are provided, one for text and another that takes a Twitter handle.

**Text Endpoint**

1. From the Application Builder Admin console, navigate to the Endpoints section and create an Endpoint named `personality_insights_text`.
2. Add a new parameter called `text`.  The sample text can be anything.
3. Copy the code from [personality-insights-text-endpoint.rb](/personality_insights/ApplicationBuilder/personality-insights-text-endpoint.rb)
4. Update the endpoint variable to point to your Bluemix application route by changing `YOUR_ENDPOINT_HERE` to use your application's route.
5. Test the endpoint.
6. Save it.

**Twitter Endpoint**

1. From the Application Builder Admin console, navigate to the Endpoints section and create an Endpoint named `personality_insights_twitter`.
2. Add a new parameter called `handle`.  The sample text can be `IBMWatson`
3. Copy the code from [personality-insights-twitter-endpoint.rb](/personality_insights/ApplicationBuilder/personality-insights-twitter-endpoint.rb)
4. Update the endpoint variable to point to your Bluemix application route by changing `YOUR_ENDPOINT_HERE` to use your application's route.
5. Test the endpoint
6. Save it.



#### Sample Proxy Configuration (for < WEX 11)
To use the proxy you must [update the Application Builder Proxy configuration](https://github.com/IBM-Watson/wex-appbuilder-sample-proxy/blob/master/config.ru) to point to your deployed Bluemix application.  Add the following line, updating the `MY_APPLICATION_ENDPOINT` to use your applications route URL.

```ruby
set :personality_insights_endpoint, "http://MY_APPLICATION_ENDPOINT.mybluemix.net/pi/"
```


Add the following code to the sample proxy.

```
post '/pi/model_text/' do
   data = JSON.load(request.body)

   url = "#{settings.personality_insights_endpoint}model_text/"
   body = { :text => data["text"] }
   headers = { "Content-Type" => "application/json" }

   response = Excon.post(url, :body => body.to_json, :headers => headers)
   response.body
end


post '/pi/model_twitter/' do
   data = JSON.load(request.body)

   url = "#{settings.personality_insights_endpoint}model_tweets/#{data["handle"]}"
   response = Excon.get(url, :headers => headers)

   response.body
end
```

Once you have updated the Proxy, restart the Application Builder server for the changes to take effect.


#### Building Personality Insights widget #1: Send Text to the Personality Insights Service

The purpose of this widget is to send a single block of text to the Personality Insights service for analysis. The results of that analysis are then displayed in the Application Builder UI.

Once you have logged into the Application Builder Administration tool, follow these steps to create the custom widget and add it to the search results page.

1. In the Application Builder Administration Tool, navigate to the Pages & Widgets -> Book Title page. (If you have different entities in your application, choose one that has a snippet or be prepared to modify the example widget code.)
2. Create a Custom new widget.
3. Set the ID of the widget to be `Watson_PI`
4. Set the title of the widget to be `Watson Personality Insights`
5. Copy and paste the [code for this widget](/personality_insights/ApplicationBuilder/personality-insights-text-widget.erb) into the Type Specific Configuration.
6. If necessary, update the line `model = endpoint("personality_insights_text", text: text_to_analyze)` to either use your endpoint name.  If not using App Builder Endpoints (pre-WEX 11), the snippet below shows how you can use the Ruby URI::HTTP class to build a request for either the Sample Proxy or Bluemix.
7. Click to turn "Asynchronously load content" on.
8. Save the widget.
9. Go back to the Book Title page
10. Add the `Watson_PI` widget to the Book Title page and save the page configuration.

At this point the widget should be fully configured.  To test the widget, navigate to the application and choose a Book Title entity. The example shown in the below image uses the title from "Lug Nuts!" to create the model displayed. 

This simple example illustrates a basic integration. You should choose text that is relevant to a user or entity that you are modeling and you must provide enough text to generate a reliable model.  See the [Personality Insights documentation](http://www.ibm.com/smarterplanet/us/en/ibmwatson/developercloud/personality-insights.html) for more guidance on text selection and recommendations.

![Screen shot of "Watson Personality Insights" widget.](/personality_insights/ApplicationBuilder/personality-insights-widget.png)

__*The "Sent Text" Personality Insights widget*__

[The widget](/personality_insights/ApplicationBuilder/personality-insights-text-widget.erb) is fully commented if you are curious about how the code works or are interested in extending the example functionality in a new widget.

##### Using Ruby URI::HTTP in a WEX < 11.0 Widget

The following snippet demostrates using Ruby's [URI::HTTP](http://ruby-doc.org/stdlib-2.2.3/libdoc/uri/rdoc/URI/HTTP.html) class.  This snippet only applies if you are ***not*** using App Builder Endpoints and using a version of WEX less than 11.0.  If you choose not to use App Builder Endpoints and you are using WEX 11+, you should at least still use the provided HTTP helper object provided by Application Builder.

```
# determine the endpoint for the proxy.
# If you've deployed the Sample Proxy to the same server as Application Builder you can...
origin = URI.parse(request.original_url)

# Build your endpoint URL
endpoint_builder = {
  :host => origin.host,
  :port => origin.port,
  :scheme => origin.scheme,
  :path => '/proxy/pi/model_text/'  # should correspond to the request handlers in your endpoint  
}

url = URI::HTTP.build(endpoint_builder)

# choose the text you want to analyze
data = {
  :text => @subject['title'].join(" ")
}.to_json

# Make the http request
req = Net::HTTP::Post.new(url.to_s)
req.body = data
req.content_type = 'content/json'
response = Net::HTTP.start(url.hostname, url.port) do |http|
  http.request(req)
end

# store the response for later use in the widget
model = response.body
```


#### Building Personality Insights Widget #2: Twitter Status Personality Insight Widget

The purpose of this widget is to demonstrate the use of an Ajax widget that gets information at run time.  For example, if you were looking at a customer page you could ask the customer for their Twitter handle, which will then allow you to create a model that might be correlated with information available from other systems accessible from the entity page within Application Builder. For the purposes of the example, we're going to add the widget to the Home page.

Once you have logged into the Application Builder Administration tool, follow these steps to create the custom widget and add it to the search results page.

1. In the Application Builder Administration Tool, navigate to the `Layouts` tab and select the `home` page.
2. Create a Custom new widget.
3. Set the ID of the widget to be `watson_twitter_pi`
4. Set the title of the widget to be `Watson Personality Insight - Twitter`
5. Copy and paste the [code for this widget](/personality_insights/ApplicationBuilder/personality-insights-twitter-widget.erb) into the Type Specific Configuration.
6. If necessary, update the `endpoint` variable in the button click handler to use the proper URL for your endpoint.  If you've been following along these docs then no changes should be required.  You will need to update the URL if you are not using App Builder Endpoints or if you changed the endpoint function name. 
7. Save the widget.
8. Go back to the Home page.
9. Add the `watson_twitter_pi` widget to the Home page and save the page configuration.

At this point the widget should be fully configured.  To test the widget, navigate to the Application Builder Home page.  Your widget should be visible.  Try entering your Twitter handle or using the IBM Watson Twitter stream, `IBMWatson`. Once you've submitted the form, your Bluemix service will fetch about 200 Twitter status updates for the provided handle. The resulting analysis from Watson will be displayed under the form in the widget. One obvious extension is creating a widget that automatically fetches Tweets based on a Twitter handle stored in an entity field rather than waiting for a user to enter a Twitter user name in the browser after the fact.


### Production and Deployment Considerations

These examples are intended for demonstrative purposes only.  While you might be able to reuse the patterns and even parts of the code from these examples, there are several concerns that should be considered when developing a production-grade application.

- _Maintainability_ - For the example, only the Watson Personality Insights Service is built into the Bluemix application. If this were a real application you should consider creating a single Bluemix application for all cloud based cognitive (or other) services used within Bluemix.
- _Security_ - The example Bluemix applications are completely open and have no security.
- _Scalability_ - This example uses only a single cloud instance with the default Bluemix application settings.  In a production scenario consider how much hardware will be required and adjust the Bluemix application settings accordingly.
- _Using Twitter_ - The Twitter API has limits in how many requests can be made to the API per hour.  Consider carefully whether your required load can be met by these limits and use the appropriate API authentication mechanism and caching strategies.
- _User Experience_ - The example widgets are only meant to demonstrate basic interaction. For a custom application using Application Builder you should carefully consider widget placement, overall look and feel, user needs, and how Watson Personality Insights can provide value to end users.
- _Integration with other Analytics_ - By itself a model is not that interesting, but there are opportunities for generating insights with further analysis. Consider how this information might be used in conjunction with other data available in your organization and what you might show an end user in Application Builder based on your insights. For example, a user portrait might only be a means to an end and only actionable recommendations are shown in Application Builder rather than a fancy visualization.
- _Performance_ - Rather than generating portraits from the Personality Insights service with every request, it may be worth considering caching the information (perhaps even in a Watson Content Explorer search collection) and updating only periodically instead of with every request.
- _Privacy_ - Use common sense and best practices when requesting and collecting information from users with the intention of generating a Personality Insights portrait. In the case of publicly available Twitter data, the need is decreased, but consider carefully corporate policies and user's privacy when collecting information from them for the purpose of generating portraits to assist with psychographic segmentation.


## Possible Use Cases for a Watson Personality Insights/Watson Explorer Integration

Watson Personality Insights offers another dimension of the "360 Degree View" of a customer or other entity and starts to paint a picture of a person not just through their data but their personality and personal preferences.  Here are some ideas to help get you started for how Watson Personality Insights might be fully integrated into a Watson Explorer application.

* Display a user's portrait on an entity page alongside other information relevant to that entity. For Person entities, consider using that person's Twitter or Facebook status updates.  For "thing" entities such as products, consider collecting Twitter status updates based on a search for that product name or a #hashtag to generate a portrait for people who discuss that product or service.
* Perform additional analysis using the generated portraits as one of the variables under consideration, for example looking for correlations and other insights across other data within your organization, such as customer tickets or sales histories.
* Display recommendations for action based on a psychographic segmentation built using portraits generated by the Personality Insights service.
* Tailor the Application Builder UI based on the user's portrait.
* Incorporate the generated model into other entity analytics available within Application Builder.
* Index portraits so they can be searched, filtered, sorted and displayed alongside other information available in your organization.
