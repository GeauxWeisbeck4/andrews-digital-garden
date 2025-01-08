---
id: 01JGFP1YK84PJPH19WCEAS9SBF
modified: 2024-12-31T20:00:46-05:00
---
# Chapter 5. Third-Party Services

# A NOTE FOR EARLY RELEASE READERS

With Early Release ebooks, you get books in their earliest form—the author’s raw and unedited content as they write—so you can take advantage of these technologies long before the official release of these titles.

This will be the 5th chapter of the final book.

If you have comments about how we might improve the content and/or examples in this book, or if you notice missing material within this chapter, please reach out to the author at _milecia.mcgregor@gmail.com_.

There will come a point in developing a product when you’ll need to use a third-party service. Third-party services offer complex or proprietary functionality that exists outside your codebase and your company. The third-party service is usually managed by another company; you pay a fee to use it and have access to support when things go wrong.

Third-party services come up when you need functionality that would take a significant amount of time to implement, would be difficult to maintain, and would likely need its own team, such as payment systems, authentication/authorization, and monitoring and logging for your app.

In this chapter, we’ll go over:

- Choosing third-party services
    
- Implementing a third-party service in your system
    
- Managing errors, outages, and upgrades
    

Working with third-party services is going to come up at some point, so it’s best to go in with an idea of what you’re getting into. Usually, the third-party service provides an API or software development kit (SDK) as an npm package that you install in your app, but you’ll need to provide some type of credentials to access the full functionality. There are also some SDKs that exist without a service just to give you specific functionality.

Keep in mind that there’s a chance that something on the service’s side could change and break without letting you know. It’s all interface code that you’ll be working with, so it won’t be any different in that regard.

We’ll be adding Stripe as the third-party service to handle the payment part of the app. First, I’ll go over some things you should consider for third-party services. Then, you’ll implement the controller, service, and other code to get Stripe working in the demo project.

# Choosing a Third-Party Service

There are numerous services to handle payments, authentication, data processing, tax handling, logging, working with other company’s data, and any other functionality you can think of. That can make the task of choosing a service seem daunting because you have to decide what to build and balance that with costs. In reality, this is pretty similar to the process of selecting a package to use in your code.

The most important difference between selecting a third-party service and a package is that the services cost money. There’s nothing wrong with selecting a paid service over a free package if it’s going to save you development time and get features out faster and with more confidence that the app will work as expected. Getting locked into a great product with a good pricing structure is something to hope for compared to the opposite.

Here are some things you might look into as you research services you need:

- Does it have good documentation that is regularly updated?
    
- How are new version releases handled?
    
- Is responsive support available?
    
- Is the pricing structure clearly defined, even if you have to get on a call with a sales rep?
    
- Are there other options for the functionality you need?
    
- Is the company mature and stable?
    
- How easy would it be to add the solution to your existing architecture?
    
- How does this solution compare to other popular solutions for this service?
    
- Is there a good sandbox environment for testing?
    
- How much effort would it take to switch to a different tool from this one?
    

These are all things to consider, and there will be some interesting trade-offs. You might even get into more industry-specific questions around legal compliance and regulations. Once you’ve done a thorough business analysis of a service, take it for a spin in your codebase. See how long it takes to add the smallest working integration to your code. That’ll be a good early test run to see what it would be like working with the service.

When you’re doing this testing, don’t forget to share your findings with the team as well as with management. Do some quick demos to show how things are going in the code. This is one way you can help others on the team level up. It’ll also help you understand the service at a deeper level because you’ll be answering questions that the team has.

While you’re checking out how the integration works, look into the reputation of the product and company. It’s important to know the experiences other developers have had with any service you’re considering paying for. If the service seems awesome and has great support, see if anyone else had that experience after they signed a contract. It may seem cynical, but it’s good to check these things out before you are deeply integrated with a product.

You also need to consider the country of origin of the service. If you develop software for any government, there will likely be countries that you have to avoid selecting software from. This is something you might not initially think about depending on what industry you’re in, but geopolitics affects the tools we use to develop software. On the other side, if your company wants to show its support for a country, you can evaluate services for good geopolitical reasons.

As you continue to research third-party services, consider looking at the GitHub repo for the service if there is one. You’ll be able to see other developers’ experiences with it and how the third-party service company handled any bugs or questions. You’ll probably find the most common issues everyone runs into before you’re too deep into the implementation.

One more thing to check for is how third-party services interact with one another. You may need to use multiple services through another service like [Zapier](https://zapier.com/) or your cloud provider’s services, which can complicate things. See if any of the services complement the other tools and services you’re using.

# List of Potential Services

It can be hard to know what to even look at when you figure out you need a third-party service. These are some options to start with:

Payment handlers

- [Stripe](https://stripe.com/docs)
    
- [Square](https://developer.squareup.com/docs)
    
- [Clover](https://docs.clover.com/docs)
    
- [PayPal](https://developer.paypal.com/home)
    
- [Paddle](https://www.paddle.com/)
    

Logging/monitoring

- [DataDog](https://docs.datadoghq.com/)
    
- [New Relic](https://docs.newrelic.com/)
    
- [Splunk](https://docs.splunk.com/Documentation)
    
- [Sentry](https://docs.sentry.io/)
    

Third-party apps

- [Meta](https://developers.facebook.com/docs/)
    
- [Instagram](https://developers.facebook.com/docs/instagram/)
    
- [YouTube](https://developers.google.com/youtube/v3)
    
- [Google Workspace](https://developers.google.com/workspace)
    

Ecommerce

- [Shopify](https://shopify.dev/docs/api)
    
- [Amazon](https://developer-docs.amazon.com/sp-api)
    
- [Etsy](https://developers.etsy.com/documentation)
    
- [BigCommerce](https://developer.bigcommerce.com/api-docs/overview)
    

Authentication

- [Auth0](https://auth0.com/docs)
    
- [FusionAuth](https://fusionauth.io/docs)
    
- [Amazon Cognito](https://docs.aws.amazon.com/cognito)
    
- [SuperTokens](https://supertokens.com/docs/guides)
    
- [Clerk](https://clerk.com/)
    

Email services

- [SendGrid](https://docs.sendgrid.com/for-developers)
    
- [Amazon Simple Email Service (SES)](https://docs.aws.amazon.com/ses/latest/dg/send-email-api.html)
    
- [Mailgun](https://documentation.mailgun.com/en/latest/quickstart.html)
    
- [Postmark](https://postmarkapp.com/email-api)
    
- [Brevo](https://www.brevo.com/)
    

Geolocation

- [Google Maps](https://developers.google.com/maps/documentation/javascript)
    
- [Mapbox](https://www.mapbox.com/)
    
- [Esri ArcGIS](https://developers.arcgis.com/javascript/latest)
    
- [Radar](https://radar.com/documentation)
    

You may run into some very niche services out there depending on what your product needs. These services can be challenging to integrate, so make sure you are clearly communicating what you’re finding and ask others to join in the search, too.

Always check the data structure that you’ll get from the API responses because some of them won’t be as well documented as others. Do some initial exploration between services you’re considering to see what you will get back. This is another good way to evaluate how well a service is going to fit into your infrastructure. You’d be surprised at the response structure you’ll get back from some of the services you’ll use over the years.

Since the app you’re building will require the ability for users to make payments, you’ll need to implement a third-party service to handle this: a service that has Payment Card Industry Data Security Standard (PCI DSS) compliance built in and has lots of security in place to keep users’ financial and personal information secure. For this part of the project, you’ll get to work with Stripe.

# Integrating Stripe

The reason you’ll use Stripe in this project is because it has great documentation (which you’ll refer to often), a large community of developers uses it, and there’s a pretty good testing environment for it. I still encourage you to take a look at some of the other options and compare the trade-offs you see.

###### NOTE

I won’t go through setting up a Stripe account because it gets pretty in-depth, and you will need to add your personal information. If you don’t feel comfortable doing that, it’s OK. This information will be provided by your organization anyway. I’ll go over the programmatic implementation so that you can follow along with the code to see what you would do after the account is set up.

The first thing you need to do is decide how you want to handle third-party services in your app. When you’re thinking about this, keep in mind that you will likely have other integrations as the product grows and needs more capabilities. The approach you’ll take in this project is adding a new folder called _integrations_ in _src_. Inside the _integrations_ folder, you’ll add a subfolder called _stripe_ with a few files in it. This will make your folder tree look like this:

|__ prisma
|__ src
|____ integrations
|______ stripe
|________ stripe.controller.spec.ts
|________ stripe.controller.ts
|________ stripe.interface.ts
|________ stripe.module.ts
|________ stripe.service.spec.ts
|________ stripe.service.ts
|____ orders
|____ app.module.ts
|____ main.ts
|__ test

This is also a great time to update your architecture diagram to show how this new service integrates with your overall app, as in [Figure 5-1](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch05.html#updated_architecture_diagram). Docs are never really finished, so remember to update relevant docs as you add new functionality. The way third-party services will work with this app includes some event handling with your cloud service using tools like [Amazon CloudWatch](https://aws.amazon.com/cloudwatch/) and [EventBridge](https://aws.amazon.com/eventbridge/) as well as directly interfacing with your own APIs.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098122249/files/assets/fsjs_0501.png)

###### Figure 5-1. Updated architecture diagram

You’ll follow the same programming pattern here as you did with the orders functionality. The endpoints your frontend will call are defined in the controller. The interactions directly with the Stripe API will be in the service. Keeping this separation of concerns is what helps make your code modular and easier to test. You’ll want to keep that up even as you add other third-party code.

You can start by updating the new _stripe.module.ts_ file to import and use the new service and controller because this is a quick win:

###### NOTE

Starting with a quick win is a way to build up momentum when you’re facing something big. It can make the problem feel more approachable. As you write out functionality, see if you like to write code in a specific order. It can help you get to the more complex details faster. Throughout this book, I’ll typically start with the initial module imports, move on to the controller, and finish with the service. I’ll update the interface as we go and save the tests for last. But I tend to jump around files _a lot_ during real development.

```
// stripe.module.ts
```

You need to add a new environment variable to _.env_ to use the Stripe API. You’ll get this value from your account, but there’s a test one you can use that’s found in the docs. So in your _.env_, add the following line.

```
STRIPE_SECRET_KEY
```

## Writing the Controller

From here, you can jump into your Stripe controller and start writing out endpoints you’ll use. You’ll need an endpoint to take user payments and an endpoint to update your products in Stripe’s system. Open _stripe.controller.ts_ and add this code to start making the endpoints:

```
// stripe.controller.ts
```

In this controller, you’ve defined one endpoint. The endpoint is how you can handle payments through Stripe. This one is a little different; you’ll need to do a redirect in your service method because the user will be sent to Stripe’s checkout page through the payment request, and you need a way to send them back to your app after the payment has been made. You can check out why you do the redirect by looking at the diagram on [how Stripe handles checkouts](https://stripe.com/docs/payments/checkout/how-checkout-works).

Keep in mind you have the `CreateStripePaymentDto` in this controller, so you need to double-check that the interface meets the exact data requirements. As the app grows, you will also have some nested validation so that you can ensure you’re sending the correct values to Stripe’s API:

```
// stripe.interface.ts
```

This validation functionality can be done with other packages like [Joi](https://joi.dev/) or [Zod](https://github.com/colinhacks/zod) in your middleware if you aren’t using NestJS. `StripePriceData` is something you may use multiple times throughout your service implementation as it grows. A good practice is to mirror the types and requirements from the service docs so that you know you’re sending exactly what they expect. Sometimes you can simply use the types that come directly from the SDK.

There are also times when you know that you’ll be using only certain fields from the types and you’ll be adding other fields that don’t come from the SDK. You’ll have to determine if it’s worth the time to fit the SDK types into what you need or if you should write your own. When you know that you’ll use the response exactly how it comes to you, go ahead and use the SDK types. If you know that you will have some new combination of fields or the SDK types need to be in a different format, consider mirroring the types to build your own interface.

###### NOTE

Not every service is as developer friendly as Stripe. If you aren’t sure what types the request expects or what the response will be, just test the service and see what you get back. Sometimes, third-party services require a discovery phase so that you know what you’re working with and how you can add the correct types to your app. You might use tools like [mitmproxy](https://mitmproxy.org/) or even Postman to see what you’ll be working with.

## Writing the Service

Now that you’ve written out the bulk of the code for the controller, turn your attention to the service where you’ll work directly with Stripe. Stripe has a nice npm package that will let you use the API.

Let’s implement the ability to make payments in Stripe. As you write the code, you can reference the [docs for the checkout API to handle payments](https://stripe.com/docs/api/checkout/sessions/create). In _stripe.service.ts_, you’ll do some file setup like importing packages and adding your constructor and logger. Then you can add the method to handle the request to the Stripe checkout API for your payments:

```
// stripe.service.ts
```

One of the biggest things to note here is the logger that’s been initialized in the service. The logs you write will be invaluable when you’re debugging issues because you can create a record of every action that happens. When you combine this with another third-party service like Datadog, the insights you get will help you fix issues with precision.

Between the logs and the error handling is the heart of third-party service integration: making sure that your code can handle any issues that come up from the third-party code. One thing to note is that third-party code might break some of your conventions, but that’s one of the trade-offs of using the service.

Something else to keep in the back of your mind are any edge cases you can think of. What will your app do if the service you need is down? It’s important to have this conversation with the Product team as you find out the quirks of the service through all your testing, which you’ll get into in [Chapter 7](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch07.html#backend_testing).

You’ll notice there’s a new field that needs to be added to the database: `stripeProductId.` You want to keep track of some third-party data in your system so that you can perform actions through the API and validate any changes. That means you’ll need to update the _schema.prisma_ and run a migration. I’ll leave that to you to do, but you can check your _schema.prisma_ against the version in the [GitHub repo](https://github.com/flippedcoder/dashboard-server/blob/main/prisma/schema.prisma).

# Conclusion

This chapter focused on getting comfortable researching and using third-party services in your code. The most important thing to remember when working with these services is that sometimes they just don’t behave the way you expect. Weird race conditions can happen, data can come back in inconsistent formats, and request parameters can change. This is where error handling and logging will be your best friends. Issues will pop up with third-party services from time to time, and that’s OK. It’s nothing you can’t handle with some communication among the dev and Product teams and a willingness to try different code strategies.