---
id: 01JH1NRBV4HR4K0GPS74FJR1XY
modified: 2025-01-07T19:41:54-05:00
---
# Chapter 19. Frontend Security Considerations

# A NOTE FOR EARLY RELEASE READERS

With Early Release ebooks, you get books in their earliest form—the author’s raw and unedited content as they write—so you can take advantage of these technologies long before the official release of these titles.

This will be the 19th chapter of the final book.

If you have comments about how we might improve the content and/or examples in this book, or if you notice missing material within this chapter, please reach out to the author at _milecia.mcgregor@gmail.com_.

Security is another area of the frontend that should be top of mind as you implement new features. Security is important enough to have its own book to get into the details, but this chapter will cover enough for you to know what to be aware of.

You’ve already learned about some of the vulnerabilities and remedies for security on the backend in [Chapter 8](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch08.html#backend_security_considerations). The frontend is usually the most accessible part of a product and can act as a gateway for server attacks. Now you have to consider things like browser vulnerabilities and ways malicious users can manipulate the flow of how the app should work to gain more access than they should have. Think about how you store and transmit data on the frontend because everything that loads in the browser is accessible by users if they just open the developer tools. You and the team need to find a balance between user convenience and security.

In this chapter, I’ll cover:

- More of the OWASP Top 10
    
- Common vulnerability vectors
    
- How to attack an app
    
- Ways attackers can get information directly from the browser
    
- Strategies to reduce the number of attack possibilities
    

You’ll need to check with any local or regional data-privacy and compliance laws and rules that your product is governed by as well, such as [PCI DSS](https://www.nerdwallet.com/article/small-business/pci-compliance#the-12-pci-compliance-requirements), [HIPAA](https://www.hipaajournal.com/hipaa-compliance-checklist/), and [GDPR](https://gdpr.eu/checklist/) (all mentioned previously in [Chapter 8](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch08.html#backend_security_considerations)). These regulations can be more strict than regular best practices, so keep that in mind.

An overlooked skill is knowing how to do some basic attacks. You don’t need to go into cybersecurity or learn a bunch of tools, but understanding what to look for will help you understand how attacks work and how you can prevent them.

# Common Vulnerabilities

Many frontend vulnerabilities can be categorized as [_insecure design_](https://owasp.org/Top10/A04_2021-Insecure_Design/). You know from [Chapter 8](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch08.html#backend_security_considerations) that the backend is more responsible for things like authentication, authorization, and the majority of security configurations. Insecure design happens _accidentally_. You and the team will do your best to ensure that the app is as secure as possible, but sometimes tight deadlines or unclear requirements can leave unexpected gaps. Let’s go through some of the commonly overlooked areas.

## Business Logic Validation

Sometimes the way an app’s user flow is designed opens it to vulnerabilities. Even the _golden path_, or the path that you and the organization behind the app would like the user to take, can leave a lot of edge cases open for someone to exploit.

In your app, one area like this could be the ordering process. What happens if a user tries to place several orders for the same product within a few minutes? Or if they cancel an order as soon as it has shipped? While these scenarios are unlikely, ignoring the possibility leaves the organization exposed to these unexpected situations.

Imagine your customers can book a customer support call on an online calendar on your app. Are there limits to how many sessions a single user can book? If not, someone could build a bot that books all the sessions, costing the organization time and money. As you go through designs and specs, look for these kinds of openings and think about how they could be exploited.

Form submission flows are common areas for attacks. It can be tempting to tell users how many attempts they have remaining or what was wrong with their submission, but be cautious with the info you provide. A malicious user could use it to build bots to automate attacks on your site. These attacks can be prevented with strategies like an unannounced limit on retries or an extra manual step, such as a CAPTCHA.

Any data that gets passed in backend requests, like the URL query string or in the body of a request, can be an [attack vector](https://owasp.org/Top10/A01_2021-Broken_Access_Control/). Such data can reveal some of the parameters used in API requests. (That isn’t necessarily a bad thing, as long as there isn’t any sensitive data in the string and the backend has its security validations in place.) Using query strings is a way you can make links shareable so that the same data can be fetched from a different tab or browser. Just be aware of what information about the API appears in the URL.

This also brings up how much data is shared with the frontend. Try to limit requested data to returning only what the user needs and be mindful of any potentially sensitive data like PII. That can include data the user doesn’t see right away but that you expect them to need soon after the request has been made to help with performance. Just because data doesn’t load on the page, that doesn’t stop people from checking network requests in the browser devtools. As you work on the frontend and add more API calls, check that they return only the data you need and that any sensitive data is provided only _after_ authentication and authorization are confirmed.

## Session Management

[_Session management_](https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html#session-expiration) deals with how you handle and store access tokens on the frontend, which can also be an [attack vector](https://owasp.org/Top10/A07_2021-Identification_and_Authentication_Failures/). Not all attacks happen from someone running bots or typing complex code into a terminal to execute remote commands. An attack can be as simple as someone checking your browser to see if there’s any authorization information in the URL or if you’re still logged in on an old device. Think about the last time _you_ had to log in to your favorite website or tool.

When you’re handling users’ PII data, you need to implement session timeouts. With [idle session timeouts](https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html#idle-timeout), the user is automatically logged out after a certain amount of time has passed and the page hasn’t triggered any new API requests. This helps prevent attackers from taking and using a session ID. The downside is that if an attacker _does_ get a valid session ID, they can keep that session active and do whatever they want. You shouldn’t store the session time on the browser because an attacker can easily modify the values. It will require some work on the backend to make sure session IDs are really invalid after the time has expired.

You should also have in place [_absolute session timeouts_](https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html#absolute-timeout): no matter where the user is logged in, after a set amount of time, they are forcibly logged out and must log in and reauthenticate to use the app. This improves on the idle session timeouts because even if a valid session ID has been hijacked, it won’t last forever. This limits how long an attacker has access to a user’s identity.

Don’t forget to have a Logout button for users to terminate their own sessions. Security-conscious users will want the ability to log out manually. With automated timeouts in place, it can be easy to overlook this. This functionality can also be added to events like the user closing the browser or page, if you want to add logout reminders.

Another option is [session renewal timeout](https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html#renewal-timeout). This is when a session ID expires and a new one is generated in the background and replaced on the frontend, without any action from the user. This allows users to stay logged in longer without giving attackers access to the same session ID. (If you’re logged into any apps where you haven’t had to reauthenticate in a while, hopefully this is what’s happening in the background.)

One more decision you have to make is if users should be able to be logged into the app on different devices at the same time. Allowing users to log in on multiple devices can potentially let an attacker log in at the same time as the user and go unnoticed for a while. If you allow this, set up monitoring and alerts to specifically identify unusual activity across sessions. On the other hand, if you allow only one login across all devices, that will require some backend work to make sure that session IDs are invalidated and that you have a mechanism to let users decide which device to stay logged in on.

## Package Version Maintenance

Package versions falling out of date is a [common vulnerability](https://owasp.org/Top10/A06_2021-Vulnerable_and_Outdated_Components/) on both the frontend and the backend. If a package has had security patches, each vulnerability in the outdated, unpatched version is well documented and publicly available on any number of [Common Weakness Enumeration (CWE) lists](https://cwe.mitre.org/data/index.html). Some attackers just go to the OWASP Top 10 itself and build a list of attacks based on those findings.

Package-version maintenance gets put off for many reasons, from not wanting to deal with breaking changes to needing to replace packages completely. I suggest having a ticket at least once a month to go through and update all your packages to the latest stable version. Ideally, you and the team can update packages as you work on new features or on eliminating other tech debt.

You can use tools like [Dependabot](https://github.com/dependabot) to flag packages with vulnerabilities directly in your repo. Other code scanner tools like [npm-audit](https://docs.npmjs.com/cli/v10/commands/npm-audit), [Snyk](https://snyk.io/), or [retire.js](https://github.com/RetireJS/retire.js) tell you which packages are out of date. Not all packages need to be maintained. You’ll find plenty that do a certain task very well but that haven’t been updated in years. With experience, you’ll learn to balance your package choices.

###### NOTE

Some of the tools mentioned here are great to integrate into your CI/CD pipeline. We’ll dive into detail on this in [Chapter 27](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch27.html#building_a_cisoliduscd_pipeline).

[Figure 19-1](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch19.html#vulnerability_report_snippet) is an example of what a report will look like if there are any vulnerabilities when you run npm-audit on your projects.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098122249/files/assets/fsjs_1901.png)

###### Figure 19-1. Vulnerability report snippet

It’s important to vet the packages you choose from the beginning. This is code that you don’t directly maintain, so you need to be able to trust that others are. If you find that one of the packages you use is no longer maintained, that could allow a vulnerability into your app. Look for replacements or consider building the functionality in house.

## Input Validation

This discussion of input validation will be slightly different from what we talked about in [Chapter 18](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch18.html#frontend_error_handling) because now you aren’t worried about what the user sees. Now you care about how the code handles what it gets from inputs. You have to programmatically [validate and sanitize](https://cheatsheetseries.owasp.org/cheatsheets/Input_Validation_Cheat_Sheet.html#client-side-vs-server-side-validation) the data you receive from users. Making sure the user has entered the correct data type is handled by any validation schemas and functions you implement in your inputs and forms.

What you need to check for is if the user is trying to force values into a certain format to get the app to do something they don’t have access to. That can include [SQL injection attacks](https://owasp.org/www-community/attacks/SQL_Injection) and [cross-site scripting (XSS) attacks](https://owasp.org/www-community/attacks/xss/). [These kinds of attacks](https://owasp.org/Top10/A03_2021-Injection/) are common because forcing values is a straightforward avenue. Your backend already has input validation and sanitization in place, but it’s nice to have some validation on the frontend to provide instant user feedback without putting any load on the server.

Anyone can type things into a form and click Submit just to see what will happen. Frontend validation and sanitization are great for UX and can slow down some basic attacks. Just always remember that they can easily be bypassed.

You can do form validation with the built-in field attributes, such as `required`, `type`, `min`, and `max`. This is one of the methods you used in your project. Here’s an example of a form using this type of validation, which you can add to your project as a form to clean up later in _src/elements/Forms/OrderForm.tsx_:

```
const
```

This will require your user to enter certain values in the different input fields before they can submit the form. You can also take a more programmatic approach to validation by using a form-handling package like React Hook Form. Here are a couple of examples of using this package with your `SearchBar` component:

```
const
```

  `{...``register``(``‘``search``’``,` `{`
    `required``:` `true``,`
    `maxLength``:` `15``,`
    `minLength``:` `3``,`
    `pattern``:` `/^[A-Za-z]+$/i``,`
  `})}`
`…`

You can use a validation schema for all of your inputs for more versatility and customization, or you can put the validations directly in the input’s props.

# Other Principles

There’s a phrase that you might hear in the cybersecurity world: “[security through obscurity](https://www.okta.com/identity-101/security-through-obscurity/).” It means that only the people who work on certain parts of the system have any knowledge about it. For example, the Security team probably knows more about roles and how authorization is implemented than you do. Keeping information about mechanisms to a need-to-know basis helps keep vulnerabilities secure. Another example of security through obscurity is having an organization-wide password tool set up so that only certain members have access to specific passwords. Everything is technically available, but it’s more secure since only the people who need access to passwords will have them.

I said this in the section on session management, but I can’t emphasize it enough: _anything_ sent from the server to the client that provides clues about your infrastructure can provide clues for malicious users looking to exploit vulnerabilities. Things you might not even think of, such as `X-Powered-By` headers and URLs that expose the hosting provider (like using default AWS API endpoints instead of using custom domains), can give away enough information to help build an attack. Be very intentional with every piece of data that is sent to the frontend because it can be easily found.

# How to Check Your Own App

Learning how to do some basic ethical hacking on your own apps will help you see security from the other side. You’ll start to see how vulnerabilities leave paths open for anyone looking. One resource I highly recommend is the [PortSwigger Web Security Academy](https://portswigger.net/web-security/all-topics). The site has a ton of in-depth articles, examples, and labs for you to play around with some common attacks. I encourage you to go through some of the labs, such as the one on [bypassing two-factor authentication (2FA)](https://portswigger.net/web-security/authentication/multi-factor/lab-2fa-simple-bypass) or one of the business logic labs, like the one on a [low-level logic flaw](https://portswigger.net/web-security/logic-flaws/examples/lab-logic-flaws-low-level).

Doing labs is just one way you can start to understand how attackers think and what they watch for when they poke around for vulnerabilities. This isn’t something you can write straightforward code for; it takes some strong observational skills, pattern finding, and a lot of creativity. That’s one of the fun parts of ethical hacking: it stretches your mind in new directions. Learning more about it will help you grow in your career.

If you decide to go even deeper into learning how to attack the apps you develop, look into the [Kali Linux OS](https://www.kali.org/get-kali/#kali-platforms). It comes with just about every tool you could need to attack any number of systems. You can even run it on a virtual machine if you want to play around with it without setting up an entire operating system. One of the best ways to know how to secure your app is understanding the tools and methods attackers will use to find and exploit any vulnerabilities that may exist in your app. You can even see what organizations are looking for by checking out bug bounties on sites like [HackerOne](https://hackerone.com/bug-bounty-programs) and [Bugcrowd](https://bugcrowd.com/engagements?category=bug_bounty&page=1&sort_by=promoted&sort_direction=desc) as well as directly on organization’s sites, such as [Microsoft](https://www.microsoft.com/en-us/msrc/bounty) and [Apple](https://security.apple.com/bounty/).

Always know the laws and keep legal restrictions in mind if you decide to try attacking a real application. You don’t want to accidentally access some organization’s database and get the attack traced to your IP address when you don’t have permission to perform that attack.

# Conclusion

In this chapter, you learned more about security on the frontend and where attackers can learn how to access user data and escalate their permissions in your systems. Attackers study the frontend systematically to figure out if there’s something they can exploit. Don’t think that your app is so small that no one will care to attack it or that it’s so well established and secure that it’s impervious to attacks. Just because not everyone knows how certain parts of the system work doesn’t mean an outside party can’t observe enough detail to figure it out. Work with the Security team to make sure the frontend is just as protected as the backend since it’s your first line of defense. Because you and your team are responsible for the full stack, you need to understand how the systems work together to keep user data safe.