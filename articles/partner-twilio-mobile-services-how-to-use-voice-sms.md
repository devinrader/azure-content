<properties linkid="develop-mobile-tutorials-twilio-for-voice-and-sms" pageTitle="Use Twilio for Voice and SMS Capabilities | Mobile Dev Center" metaKeywords="" description="Learn how to perform common tasks using the Twilio API with Windows Azure Mobile Services." metaCanonical="" services="" documentationCenter="Mobile" title="How to use Twilio for voice and SMS capabilities from Mobile Services" authors=""  solutions="" writer="twilio" manager="" editor=""  />


<h1>How to use Twilio for voice and SMS capabilities from Mobile Services</h1>

This topic shows you how to perform common tasks using Twilio and Windows Azure Mobile Services. In this tutorial you will learn how to create scripts that use Twilio to initiate a phone call and to send a Short Message Service (SMS) message. 

<h2><a id="WhatIs"></a>What is Twilio?</h2>
Twilio is powering the future of business communications, enabling developers to embed voice, VoIP, and messaging into applications. They virtualize all of the infrastructure needed in a cloud-based, global environment, exposing it through the Twilio communications API platform. Applications are simple to build and scalable. Enjoy flexibility with pay-as-you go pricing, and benefit from cloud reliability.

**Twilio Voice** allows your applications to make and receive phone calls. **Twilio Messaging** enables your applications to send and receive SMS and MMS messages. **Twilio Client** allows you to make VoIP calls from any phone, tablet, or browser and supports WebRTC.

<h2><a id="Pricing"></a>Twilio Pricing and Special Offers</h2>
Windows Azure customers receive a [special offer][special_offer]: complimentary $10 of Twilio Credit when you upgrade your Twilio Account. This Twilio Credit can be applied to any Twilio usage ($10 credit equivalent to sending as many as 1,000 SMS messages or receiving up to 1000 inbound Voice minutes, depending on the location of your phone number and message or call destination). Redeem this Twilio credit and get started at [ahoy.twilio.com/azure][special_offer].

Twilio is a pay-as-you-go service. There are no set-up fees and you can close your account at any time. You can find more details at [Twilio Pricing][twilio_pricing].  

<h2><a id="Concepts"></a>Twilio Development Concepts</h2>
Twilio uses two common web development techniques to let your application interact with the telephone network: a REST API for initiating outbound phone calls or messages, managing phone numbers and accessing account data, and a webhook to notify you when inbound communication happens.

![twilio how it works image]()

The Twilio REST API lets your application initiate outbound phone calls or send a messages.  Idiomatic client libraries are available for a variety of programming languages; for a list, see [Twilio API Libraries] [twilio_libraries].  Tutorials are available for using the Twilio any Windows Azure application written in [.NET][azure_twilio_howto_dotnet], [node.js][azure_twilio_howto_node], [Java][azure_twilio_howto_java], [PHP][azure_twilio_howto_php], [Python][azure_twilio_howto_python] or [Ruby][azure_twilio_howto_ruby].

For additional information about the Twilio REST API, see [Twilio API][twilio_api].

A Twilio web hook is a simple HTTP request Twilio makes to a URL that you have specified.  This request allows you to be notified when someone makes a phone call or sends a text message to your Twilio phone number.  The web application at the URL you specify can return TwiML, set of XML-based instructions that inform Twilio of how to proceed with a phone call or respond to a message.  

TwiML is composed of a set of commands called Verbs, some of which are listed below.  

* **&lt;Say&gt;**: Converts text to speech that is read to a caller.
* **&lt;Play&gt;**: Plays an audio file to a caller.
* **&lt;Message&gt;**: Sends an SMS or MMS message.
* **&lt;Dial&gt;**: Connects the caller to another phone number, VoIP client or Conference.
* **&lt;Gather&gt;**: Collects numeric digits entered on the telephone keypad.
* **&lt;Record&gt;**: Records the caller's voice and returns a URL of a file that contains the recording.
* **&lt;Reject&gt;**: Rejects an incoming call to your Twilio number without billing you

As an example, the following TwiML would convert the text **Hello World** to speech and read it to the caller.

    <?xml version="1.0" encoding="UTF-8" ?>
    <Response>
       <Say>Hello World</Say>
    </Response>

Learn about the other verbs their attributes and capabilities via [Twilio Markup Language documentation](http://www.twilio.com/docs/api/twiml).

<h2><a id="CreateAccount"></a>Create a Twilio Account</h2>
When you're ready to get a Twilio account, sign up at [Try Twilio][try_twilio]. You can start with a free account, and upgrade your account later.

When you sign up for a Twilio account, you'll receive an account ID and an authentication token. Both will be needed to use the REST API. To prevent unauthorized access to your account, keep your authentication token secure. Your account ID and authentication token are viewable at the [Twilio account page][twilio_account], in the fields labeled **ACCOUNT SID** and **AUTH TOKEN**, respectively.

![twilio account dashboard]()

<h2><a id="VerifyPhoneNumbers"></a>Phone Numbers</h2>
When you sign up for a Twilio account, Twilio will provide you with your very own Twilio phone number and will require you to verify a phone number that you own, like your cell phone number.

A trial Twilio account will allow you to send text messages from your Twilio phone number to your verified phone number and make outgoing phone calls from your Twilio phone number to your verified phone number.

Upgraded accounts have these restrictions removed, allowing you to send messages and make phone calls to virtually any phone number.

<h2><a id="create_app"></a>Create a Twilio-enabled Mobile Service</h2>
Creating a Twilio-enabled Mobile Service is no different that creating a standard Mobile Service.  For information on creating mobile service, see [Getting Started with Mobile Services](http://www.windowsazure.com/en-us/develop/mobile/tutorials/get-started/).

Once you've set up your Mobile Services application you can begin to Twilio-enable it which is done by adding a reference using the Twilio node.js library.   This library wraps various aspects of Twilio to provide simple and easy ways to interact with the Twilio REST API and to generate TwiML responses.  To add the library you will use Mobile Services npm module support and add the library as a dependency.

Start by enabling source control for your Mobile Service. If you've never set up source control on a Mobile Service before, the tutorial [Store Scripts in Source Control](http://www.windowsazure.com/en-us/develop/mobile/tutorials/store-scripts-in-source-control/) walks you through setting it up for the first time.

Once you have set up source control for your Mobile Service and cloned the repository to your local machine, locate the _package.json_ file in the service directory:

![package.json file located in the defaultMobile Services repository](./media/partner-twilio-mobile-services-how-to-use-voice-sms/)

The _package.json_ file allows you to configure metadata about your Mobile Services node.js project, including any project dependencies.  Specifying dependencies in this file allows you to avoid having to commit the library package into your repository.

Add Twilio as a project dependency, save the package.json file and push the change to your remote repository.  When pushed, Mobile Services will automatically identify the project dependencies and install them.

    sample showing twilio depdendency in package.json file

Now that the Twilio library is installed, you can reference and use the Twilio library in your Mobile Services scripts.

<h2><a id="howto_make_call"></a>How to: Make an outgoing call</h2>
The following script shows how to initiate an outgoing call from your Mobile Service using the **makeCall** function. This code also uses Twimlet to return the TwiML that Twilio will execute once the phone call is answered.

Substitute the **[FROM]** and **[TO]** placeholders with your provided Twilio phone number and the your verified phone number.

    var twilio = require('twilio');

    exports.post = function(request, response) {

        var client = new twilio.RestClient('[ACCOUNT_SID]', 'AUTH_TOKEN');

        client.makeCall({
            to:'[TO]', 
            from: '[FROM]',
            url: 'http://twimlets.com/message?Message%5B0%5D=ahoy%20hoy!%20This%20message%20was%20sent%20using%20Twilio%20and%20node.js&' 

        }, function(err, call) {

            // The "error" variable will contain error information, if any.
            // If the request was successful, this value will be "false"
            if (!error) {
                console.log('Success! The SID for this call is: ' + call.sid);
                console.log('Message sent on: ' + call.dateCreated);
            }
            else {
                console.log('Oops! There was an error.');
            }
            response.send(200, { error : error } );
        });
    };

For more information about the parameters passed in to the **client.makeCall** function, see [Making Calls](http://www.twilio.com/docs/api/rest/making-calls) in the Twilio documentation.

As mentioned, this code uses a Twimlet to return the TwiML response. You could instead use your own website to generate and return TwiML response. For more information, see [How to: Provide TwiML responses from your own web site](#howto_provide_twiml_responses).

<h2><a id="howto_send_sms"></a>How to: Send an SMS message</h2>
The following code shows how to send an SMS message using the **sendSms**  function. 

Substitute the **[FROM]** and **[TO]** placeholders with your provided Twilio phone number and the your verified phone number.

    var twilio = require('twilio');

    exports.post = function(request, response) {

        var client = new twilio.RestClient('[ACCOUNT_SID]', 'AUTH_TOKEN');
 
        client.sendMessage({
            to:'[TO]',
            from:'[FROM]',
            body:'ahoy hoy! This message was sent using Twilio and node.js'
        }, function(error, message) {
    
            // The "error" variable will contain error information, if any.
            // If the request was successful, this value will be "false"
            if (!error) {
                console.log('Success! The SID for this SMS message is: ' + message.sid);
                console.log('Message sent on: ' + message.dateCreated);
            }
            else {
                console.log('Oops! There was an error.');
            }
            response.send(200, { error : error } );
        });
    };

For more information about the parameters passed in to the client.sendMessage function, see  [Sending Messages](https://www.twilio.com/docs/api/rest/sending-messages) in the Twilio documentation.

<h2><a id="howto_provide_twiml_responses"></a>How to: Provide TwiML responses from your own web site</h2>

As described in [Twilio Concepts](), TwiML is a simple XML-based set of instructions that allow your application to tell Twilio what to do when with a live phone call or how to respond to a text message.  

So far each of the examples in this article have used a Twimlet, which is a Twilio provided website, to generate TwiML instructions.  However any website, including a Mobile Service, can be used to generate and return TwiML.

<div class="dev-callout">
<b>Note</b>
<p>While TwiML is designed for use by web services, you can view the TwiML in your browser. For example, click <a href="http://twimlets.com/message">twimlet_message_url</a> to see an empty &lt;Response&gt; element; as another example, click <a href="http://twimlets.com/message?Message%5B0%5D=Hello%20World">twimlet_message_url_hello_world</a> to see a &lt;Response&gt; element that contains a &lt;Say&gt; element.</p>
</div>

For example, following node.js script generates the same TwiML that was used in the example [How to make a call]().

    var twilio = require('twilio');

    exports.post = function(request, response) {
        var resp = new twilio.TwimlResponse();
        resp.say({voice:'woman'}, 'ahoy hoy! This message was sent using Twilio and node.js');
        response.set('Content-Type', 'text/xml');
        response.send(200, resp.toString());
    };

For more information about TwiML, see [https://www.twilio.com/docs/api/twiml](https://www.twilio.com/docs/api/twiml).

Once you have set up a way to provide TwiML responses, you can pass that URL into the **client.makeCall** method as shown in the following code sample:
    
    var twilio = require('twilio');

    exports.post = function(request, response) {

        var client = new twilio.RestClient('[ACCOUNT_SID]', 'AUTH_TOKEN');

        client.makeCall({
            to:'+16515556677', 
            from: '+14506667788',
            url: 'http://<your_mobile_service>.azure-mobile.net/api/ahoyhoy' 

        }, function(err, responseData) {

              // The "error" variable will contain error information, if any.
            // If the request was successful, this value will be "false"
            if (!error) {
                console.log('Success! The SID for this call is: ' + call.sid);
                console.log('Message sent on: ' + call.dateCreated);
            }
            else {
                console.log('Oops! There was an error.');
            }
            response.send(200, { error : error } );
        });
    };

[WACOM.INCLUDE [twilio_additional_services_and_next_steps](../includes/twilio_additional_services_and_next_steps.md)]


[twilio_rest_making_calls]: http://www.twilio.com/docs/api/rest/making-calls

[twilio_pricing]: http://www.twilio.com/pricing
[special_offer]: http://ahoy.twilio.com/azure
[twilio_libraries]: https://www.twilio.com/docs/libraries
[twiml]: http://www.twilio.com/docs/api/twiml
[twilio_api]: http://www.twilio.com/api
[try_twilio]: https://www.twilio.com/try-twilio
[twilio_account]:  https://www.twilio.com/user/account
[verify_phone]: https://www.twilio.com/user/account/phone-numbers/verified#


[azure_twilio_howto_dotnet]: /en-us/develop/net/how-to-guides/twilio-voice-and-sms-service/
[azure_twilio_howto_java]: /en-us/develop/java/how-to-guides/twilio-voice-and-sms-service/
[azure_twilio_howto_node]: /en-us/develop/nodejs/how-to-guides/twilio-voice-and-sms-service/
[azure_twilio_howto_ruby]: /en-us/develop/ruby/how-to-guides/twilio-voice-and-sms-service/
[azure_twilio_howto_python]: /en-us/develop/python/how-to-guides/twilio-voice-and-sms-service/
[azure_twilio_howto_php]: /en-us/develop/php/how-to-guides/twilio-voice-and-sms-service/
