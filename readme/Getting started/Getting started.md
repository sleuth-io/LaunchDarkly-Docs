---
title: "Getting started"
excerpt: ""
---
Welcome to LaunchDarkly! We're excited to partner with you as you use feature flags to make high-impact changes with minimal risk and maximum control.

It only takes a few minutes to get started with LaunchDarkly and control your first feature flag.
[block:api-header]
{
  "title": "Integrating with LaunchDarkly"
}
[/block]
The steps to integrate your application with LaunchDarkly are common across all SDKs. We provide a variety of client, server, and mobile SDKs for you to use. To learn more about our SDK offerings, read [Choosing an SDK](#section-choosing-an-sdk).

To set up LaunchDarkly:

1. **Install the LaunchDarkly SDK in your application using your project's dependency manager.** This lets your application access the LaunchDarkly SDK.

2. **Import the LaunchDarkly client in your application code.** This client is the primary way your application uses the SDK and communicates with LaunchDarkly.

3. **Configure the LaunchDarkly client with the appropriate key for your environment and create the client.** Your SDK key, Client-side ID, and Mobile key uniquely identify your project and environment, and they authorize your application to connect to LaunchDarkly.

4. **Set which feature flag variation a user will see.** Every feature flag is uniquely identified by a feature flag key. Use the key to associate flag variations with different users.
[block:callout]
{
  "type": "info",
  "title": "Adding users to LaunchDarkly",
  "body": "You don't have to send users to LaunchDarkly in advance. You can target them with feature flags have LaunchDarkly accounts of their own. \n\nUsers appear in the dashboard automatically after they encounter feature flags."
}
[/block]
5. **Shut down the LaunchDarkly client when your application is about to terminate.** The client releases the resources it is using and sends pending analytics events to LaunchDarkly. If your application quits without this shutdown step, you may not see requests and users on the dashboard, because they are derived from analytics events. **You only need to shut down the application manually once**.
[block:api-header]
{
  "title": "Choosing an SDK"
}
[/block]
We have a variety of client-side, server-side, and mobile SDKs to choose from. 

To learn more about the different types of SDKs, read [Client-side and server-side SDKs](doc:client-side-and-server-side).

Explore the following SDK reference guides for specific details about how to use LaunchDarkly with your tech stack. To learn more about advanced configuration options for your platform's SDK, read the Reference guides section.

Server-side SDKs:

* [.NET](doc:dotnet-sdk-reference)
* [C/C++ (server-side)](doc:c-server-sdk-reference)
* [Go](doc:go-sdk-reference)
* [Java](doc:java-sdk-reference)
* [Node.js (server-side)](doc:node-sdk-reference)
* [PHP](doc:php-sdk-reference)
* [Python](doc:python-sdk-reference)
* [Ruby](doc:ruby-sdk-reference)

Client-side SDKs:

* [Android](doc:android-sdk-reference)
* [C/C++ (client-side)](doc:c-sdk-reference)
* [Electron](doc:electron-sdk-reference)
* [iOS (Objective-C)](doc:ios-objc-sdk-reference)
* [iOS (Swift)](doc:ios-sdk-reference)
* [JavaScript](doc:js-sdk-reference)
* [Node.js (client-side)](doc:node-client-sdk-reference)
* [React](doc:react-sdk-reference)
* [React Native](doc:react-native-sdk-reference)
* [Roku](doc:roku-sdk-reference) 
* [Xamarin](doc:xamarin-sdk-reference)

### Flag evaluations are always available
If the SDK you're using loses the connection with LaunchDarkly, your feature flags will still work. The SDK relies on its stored state to evaluate flags. 

By default, an SDK initializes with an empty state. When the SDK first initializes, it opens a streaming connection to LaunchDarkly. The response from LaunchDarkly contains the SDKs' current state, which your SDK uses to make any necessary changes to feature flag. After the initial update, the SDK keeps a streaming connection open to LaunchDarkly. If you make a change in the LaunchDarkly dashboard or with the REST API, LaunchDarkly sends these changes to all connected SDKs automatically.

If the SDK ever loses connectivity to LaunchDarkly, it continues to try to establish a streaming connection until it succeeds. If you evaluate a flag before the SDK receives its initial state, or you try to fetch a flag which otherwise doesn't exist, then the SDK returns a fallback value you can specify in the flag's settings. 

All SDKs provide synchronous and asynchronous ways of waiting for the SDKs state to initialize.
[block:api-header]
{
  "title": "Next steps"
}
[/block]
That's it! Now you can manage features on your dashboard-- read on to see how to [target users](doc:targeting-users) or do [controlled rollouts](doc:targeting-users) and [A/B tests](doc:running-ab-tests) without re-deploying your application!

Once you're ready to test your integration, head over to the [debugger](doc:debugger) to get a real-time view of your feature flag requests as they're received.