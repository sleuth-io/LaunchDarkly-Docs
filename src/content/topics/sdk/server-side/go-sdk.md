---
title: "Go SDK Reference"
excerpt: ""
---
This reference guide documents all of the methods available in our Go SDK, and explains in detail how these methods work. If you want to dig even deeper, our SDKs are open source-- head to our [Go SDK GitHub repository](https://github.com/launchdarkly/go-server-sdk). The online [Go API docs](https://godoc.org/github.com/launchdarkly/go-server-sdk) contain the programmatic definitions of every type and method. Additionally you can clone and run a [sample application](https://github.com/launchdarkly/hello-go) using this SDK.
## Getting started
Building on top of our [Quickstart](./getting-started) guide, the following steps will get you started with using the LaunchDarkly SDK in your Go application.

The first step is to install the LaunchDarkly SDK as a dependency in your application using your application's dependency manager. Refer to the [SDK releases page](https://github.com/launchdarkly/go-server-sdk/releases) to identify the latest major version.
[block:code]
{
  "codes": [
    {
      "code": "go get gopkg.in/launchdarkly/go-server-sdk.v4",
      "language": "go"
    }
  ]
}
[/block]
Next you should import the LaunchDarkly client in your application code.
[block:code]
{
  "codes": [
    {
      "code": "import ld \"gopkg.in/launchdarkly/go-server-sdk.v4\"",
      "language": "go"
    }
  ]
}
[/block]
Once the SDK is installed and imported, you'll want to create a single, shared instance of the LaunchDarkly client. You should specify your SDK key here so that your application will be authorized to connect to LaunchDarkly and for your application and environment.

We'll assume you've imported the LaunchDarkly SDK package as `ld`:
[block:code]
{
  "codes": [
    {
      "code": "ldClient, _ := ld.MakeClient(\"YOUR_SDK_KEY\", 5 * time.Second)",
      "language": "go"
    }
  ]
}
[/block]
The timeout parameter (the second argument to `MakeClient`) can be used to block initialization until the client has been bootstrapped. 
<Callout intent="info">
<CalloutTitle>Best practices for error handling</CalloutTitle>
   <CalloutDescription>The second return type in these code samples (` _ `) represents an error in case the LaunchDarkly client does not initialize. Consider naming the return value and using it with proper error handling.</CalloutDescription>
</Callout>

<Callout intent="alert">
  <CalloutTitle>LDClient must be a singleton</CalloutTitle>
   <CalloutDescription>It's important to make this a singleton-- internally, the client instance maintains internal state that allows us to serve feature flags without making any remote requests. **Be sure that you're not instantiating a new client with every request.**</CalloutDescription>
</Callout>

Using `ldClient`, you can check which variation a particular user should receive for a given feature flag.
[block:code]
{
  "codes": [
    {
      "code": "key := \"user@test.com\"\nshowFeature, _ := ldClient.BoolVariation(\"your.flag.key\", ld.NewUser(key), false)\nif showFeature {\n  // application code to show the feature\n} else {\n  // the code to run if the feature is off \n}",
      "language": "go"
    }
  ]
}
[/block]
Lastly, when your application is about to terminate, shut down `ldClient`. This ensures that the client releases any resources it is using, and that any pending analytics events are delivered to LaunchDarkly. If your application quits without this shutdown step, you may not see your requests and users on the dashboard, because they are derived from analytics events. **This is something you only need to do once**.
[block:code]
{
  "codes": [
    {
      "code": "// shut down the client, since we're about to quit\nldClient.Close()",
      "language": "go"
    }
  ]
}
[/block]

## Customizing your client
You can also pass custom parameters to the client by creating a custom configuration object. It's easiest to create one by starting with the `DefaultConfig` and assigning to the fields you need to customize:
[block:code]
{
  "codes": [
    {
      "code": "config := ld.DefaultConfig\nconfig.FlushInterval = 
1. * time.Second\nldClient := ld.MakeCustomClient(\"YOUR_SDK_KEY\", config, 5 * time.Second)",
      "language": "go"
    }
  ]
}
[/block]
Here, we've customized the client flush interval parameter. See the [Godocs](https://godoc.org/gopkg.in/launchdarkly/go-server-sdk.v4#Config) for a complete list of configuration options for the client.
## Users
Feature flag targeting and rollouts are all determined by the *user* you pass to your `Variation` calls. The Go SDK defines a `User` struct to make this easy. Here's an example:
[block:code]
{
  "codes": [
    {
      "code": "key := \"aa0ceb\"\nfirstName := \"Ernestina\"\nlastName := \"Evans\"\nemail := \"ernestina@example.com\"\ncustom := map[string]interface{} {\n  \"groups\": []string{\"Google\", \"Microsoft\"},\n}\nuser := ld.User{\n  Key: &key,\n  FirstName: &firstName,\n  LastName: &lastName,\n  Email: &email,\n  Custom: &custom,\n}",
      "language": "go"
    }
  ]
}
[/block]
Let's walk through this snippet. The most attribute is the user's key-- in this case we've used the hash `"aa0ceb"`. **The user key is the only mandatory user attribute**. The key should also uniquely identify each user. You can use a primary key, an e-mail address, or a hash, as long as the same user always has the same key. We recommend using a hash if possible.

All of the other attributes (like `FirstName`, `Email`, and the `Custom` attributes) are optional. The attributes you specify will automatically appear on our dashboard, meaning that you can start segmenting and targeting users with these attributes. 

Besides the `Key`, LaunchDarkly supports the following attributes at the "top level". Remember, all of these are optional:

* `Ip`: Must be an IP address.
* `FirstName`: Must be a string. If you provide a first name, you can search for users on the Users page by name.
* `LastName`: Must be a string. If you provide a last name, you can search for users on the Users page by name.
* `Country`: Must be a string representing the country associated with the user. 
* `Email`: Must be a string representing the user's e-mail address. If an `avatar` URL is not provided, we'll use [Gravatar](http://en.gravatar.com/) to try to display an avatar for the user on the Users page.
* `Avatar`: Must be an absolute URL to an avatar image for the user. 
* `Name`: Must be a string. You can search for users on the User page by name
* `Anonymous`: Must be a boolean. See the section below on anonymous users for more details.

In addition to these, you can pass us any of your own user data by passing custom attributes, like the `groups` attribute in the example above. 
<Callout intent="info">
  <CalloutTitle>A note on types</CalloutTitle>
   <CalloutDescription>Most of our built-in attributes (like names and e-mail addresses) expect string values. Custom attributes values can be strings, booleans (like true or false), numbers, or lists of strings, booleans or numbers. This is why `Custom` is defined as a `map` from `string` to `interface{}`.
If you enter a custom value on our dashboard that looks like a number or a boolean, it'll be interpreted that way. The Go SDK is strongly typed, so be aware of this distinction.</CalloutDescription>
</Callout>

Custom attributes are one of the most powerful features of LaunchDarkly. They let you target users according to any data that you want to send to us-- organizations, groups, account plans-- anything you pass to us becomes available instantly on our dashboard.
## Private user attributes
You can optionally configure the Go SDK to treat some or all user attributes as [Private user attributes](https://docs.launchdarkly.com/v2.0/docs/private-user-attributes). Private user attributes can be used for targeting purposes, but are removed from the user data sent back to LaunchDarkly.

In the Go SDK there are two ways to define private attributes for the entire LaunchDarkly client:
- In the LaunchDarkly config, you can set `AllAttributesPrivate` to true. If this is enabled, all user attributes (except the key) for all users are removed before the user is sent to LaunchDarkly.
- In the LaunchDarkly config object, you can define a list of `PrivateAttributeNames`. If any user has a custom or built-in attribute named in this list, it will be removed before the user is sent to LaunchDarkly.

You can also define a set of `PrivateAttributes` on the user object itself. For example:
[block:code]
{
  "codes": [
    {
      "code": "key := \"aa0ceb\"\nfirstName := \"Ernestina\"\nlastName := \"Evans\"\nemail := \"ernestina@example.com\"\ncustom := map[string]interface{} {\n  \"groups\": []string{\"Google\", \"Microsoft\"},\n}\nprivateAttributeNames := []string{\"email\"}
user := ld.User{\n  Key: &key,\n  FirstName: &firstName,\n  LastName: &lastName,\n  Email: &email,\n  Custom: &custom,\n  PrivateAttributes: privateAttributeNames,\n}",
      "language": "go"
    }
  ]
}
[/block]

## Anonymous users
You can also distinguish logged-in users from anonymous users in the SDK, as follows:
[block:code]
{
  "codes": [
    {
      "code": "key := \"aa0ceb\"\nanonymous := true
user := ld.User{\n  Key: &key,\n  Anonymous: &anonymous,\n}",
      "language": "go"
    }
  ]
}
[/block]
You will still need to generate a unique key for anonymous users-- session IDs or UUIDs work best for this. 

Anonymous users work just like regular users, except that they won't appear on your Users page in LaunchDarkly. You also can't search for anonymous users on your Features page, and you can't search or autocomplete by anonymous user keys. This is actually a good thing-- it keeps anonymous users from polluting your Users page!
## Variation
The `Variation` method determines whether a flag is enabled or not for a specific user.  In Go, there is a `Variation` method for each type (e.g. `BoolVariation`, `StringVariation`):
[block:code]
{
  "codes": [
    {
      "code": "shouldShow, _ := ldClient.BoolVariation(\"your.feature.key\", user, false)",
      "language": "go"
    }
  ]
}
[/block]
`Variation` calls take the feature flag key, a `User`, and a default value.

The default value will only be returned if an error is encountered-- for example, if the feature flag key doesn't exist or the user doesn't have a key specified. 

The `Variation` call will automatically create a user in LaunchDarkly if a user with that user key doesn't exist already. There's no need to create users ahead of time (but if you do need to, take a look at Identify). 
## VariationDetail
The `VariationDetail` methods (`BoolVariationDetail`, etc.) work the same as `Variation`, but also provide additional "reason" information about how a flag value was calculated (for instance, if the user matched a specific rule). You can examine the "reason" data programmatically; you can also view it with data export, if you are capturing detailed analytics events for this flag.

For more information, see [Evaluation reasons](./evaluation-reasons).
## Track
The `Track` method allows you to record actions your users take on your site. This lets you record events that take place on your server. In LaunchDarkly, you can tie these events to goals in A/B tests. Here's a simple example:
[block:code]
{
  "codes": [
    {
      "code": "ldClient.Track(\"your-goal-key\", user)",
      "language": "go"
    }
  ]
}
[/block]
You can also attach custom data to your event by passing an extra parameter to `Track`: 
[block:code]
{
  "codes": [
    {
      "code": "data := map[string]interface{}{\"price\": 320}\nldClient.Track(\"Completed purchase\", user, data)",
      "language": "go"
    }
  ]
}
[/block]
You can attach any data that can be marshaled to JSON to your events.
## Identify
The `Identify` method creates or updates users on LaunchDarkly, making them available for targeting and autocomplete on the dashboard. In most cases, you won't need to call `Identify`-- calling `Variation` will automatically create users on the dashboard for you. `Identify` can be useful if you want to pre-populate your dashboard before launching any features. 
[block:code]
{
  "codes": [
    {
      "code": "ldClient.Identify(user)",
      "language": "go"
    }
  ]
}
[/block]

## All flags

<Callout intent="alert">
 <CalloutTitle>Creating users</CalloutTitle>
   <CalloutDescription>Note that unlike variation and identify calls, AllFlagsState does not send events to LaunchDarkly. Thus, users are not created or updated in the LaunchDarkly dashboard.",
 </CalloutDescription>
</Callout>

The `AllFlagsState` method captures the state of all feature flag keys with regard to a specific user. This includes their values, as well as other metadata.

This method can be useful for passing feature flags to your front-end. In particular, it can be used to provide bootstrap flag settings for our [JavaScript SDK](./js-sdk-reference).
[block:code]
{
  "codes": [
    {
      "code": "state := ldClient.AllFlagsState(user)",
      "language": "go"
    }
  ]
}
[/block]

## Logging
The Go SDK uses Go's built in [log](https://golang.org/pkg/log/). All loggers are namespaced under `[LaunchDarkly]`. A custom logger may be passed to the SDK via the configurable `Logger` property:

[block:code]
{
  "codes": [
    {
      "code": "config := ld.DefaultConfig\nconfig.Logger = myLogger\nldClient := ld.MakeCustomClient(\"YOUR_SDK_KEY\", config, 5 * time.Second)",
      "language": "go"
    }
  ]
}
[/block]

## Secure mode hash
The `SecureModeHash` method computes an HMAC signature of a user signed with the client's SDK key. If you're using our [JavaScript SDK](./js-sdk-reference) for client-side flags, this method generates the signature you need for secure mode.
[block:code]
{
  "codes": [
    {
      "code": "ldClient.SecureModeHash(user)",
      "language": "go"
    }
  ]
}
[/block]

## Flush
Internally, the LaunchDarkly SDK keeps an event buffer for `track` and `identify` calls. These are flushed periodically in a background thread. In some situations (for example, if you're testing out the SDK on the command line), you may want to manually call `flush` to process events immediately. 
[block:code]
{
  "codes": [
    {
      "code": "ldClient.Flush();",
      "language": "go"
    }
  ]
}
[/block]

Note that the flush interval is configurable-- if you need to change the interval, you can do so by making a custom client configuration
## Offline mode
In some situations, you might want to stop making remote calls to LaunchDarkly and fall back to default values for your feature flags. For example, if your software is both cloud-hosted and distributed to customers to run on premise, it might make sense to fall back to defaults when running on premise. You can do this by setting `Offline` mode in the client's `Configuration`.
[block:code]
{
  "codes": [
    {
      "code": "config := ld.DefaultConfig\nconfig.Offline = true\nldClient, _ := ld.MakeCustomClient(\"YOUR_SDK_KEY\", config, 5 * time.Second)\nldClient.BoolVariation(\"any.feature.flag\", user, false) // will always return the default value (false)
",
      "language": "go"
    }
  ]
}
[/block]

## Logging
The Go SDK uses Go's built-in [log](https://golang.org/pkg/log) package. All loggers are namespaced under `[LaunchDarkly]`. A custom logger may be passed to the SDK via the configurable `Logger` property:
[block:code]
{
  "codes": [
    {
      "code": "config := ld.DefaultConfig\nconfig.Logger = myLogger\nldClient, _ := ld.MakeCustomClient(\"YOUR_SDK_KEY\", config, 5 * time.Second)",
      "language": "go"
    }
  ]
}
[/block]
Be aware of two considerations when enabling the DEBUG log level:

1. Debug-level logs can be very verbose. It is not recommended that you turn on debug logging in high-volume environments.
2. Potentially sensitive information is logged including LaunchDarkly users created by you in your usage of this SDK.
## Redis

The Go SDK provides an option to use Redis as a persistent store of feature flag configurations. Here's an example client configuration hooked up to a Redis instance:
[block:code]
{
  "codes": [
    {
      "code": "host = \"localhost\"\nport = 6379\ntimeout = 2 * time.Second\nstore := NewRedisFeatureStore(host, port, \"\", timeout, myLogger)
config := ld.DefaultConfig\nconfig.FeatureStore = store\nldClient, _ := ld.MakeCustomClient(\"YOUR_SDK_KEY\", config, 5 * time.Second)",
      "language": "go"
    }
  ]
}
[/block]
For advanced `RedisFeatureStore` configuration options, please see the SDK's [GoDoc](https://godoc.org/gopkg.in/launchdarkly/go-server-sdk.v4/redis).
## HTTPS Proxy
Go's standard HTTP library provides a built-in HTTPS proxy. If the HTTPS_PROXY environment variable is present then the SDK will proxy all network requests through the URL provided.

How to set the HTTPS_PROXY environment variable on Mac/Linux systems:
[block:code]
{
  "codes": [
    {
      "code": "export HTTPS_PROXY=https://web-proxy.domain.com:8080",
      "language": "shell",
      "name": "shell"
    }
  ]
}
[/block]
How to set the HTTPS_PROXY environment variable on Windows systems:
[block:code]
{
  "codes": [
    {
      "code": "set HTTPS_PROXY=https://web-proxy.domain.com:8080",
      "language": "text",
      "name": "cmd"
    }
  ]
}
[/block]