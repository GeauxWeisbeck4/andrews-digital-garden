---
id: 01JGFP52N98XDNSJQ8PVYDV5VP
modified: 2024-12-31T20:03:11-05:00
---
# Chapter 8. Backend Security Considerations

# A NOTE FOR EARLY RELEASE READERS

With Early Release ebooks, you get books in their earliest form—the author’s raw and unedited content as they write—so you can take advantage of these technologies long before the official release of these titles.

This will be the 8th chapter of the final book.

If you have comments about how we might improve the content and/or examples in this book, or if you notice missing material within this chapter, please reach out to the author at _milecia.mcgregor@gmail.com_.

The security of your app is going to be one of the top concerns for the dev team, the Product team, and the whole organization. You never want an unintended entity to gain access they aren’t supposed to have. That’s why you’ll need to look at the app from all angles, such as authentication, authorization, validation, common attacks, and external dependencies.

Security is a topic with so much depth and breadth that entire books are dedicated to it. I’m going to keep this chapter focused on what you can do on the backend specifically, but there are far more areas covered by security. You may be lucky enough to work with a Security team. They go through every part of the company’s technical infrastructure, all the way down to what you can install on your laptop.

In this chapter, you will learn about:

- Authentication methods and when to use them
    
- Authorization for users to give them different levels of access to the functionality and data in the app
    
- Why you should typically go with an out-of-the-box solution
    
- The parts of the Open Web Application Security Project (OWASP) Top 10 that apply to the backend
    
- Best security practices, regulations, and where to learn more about different areas of security
    

I’m just touching the surface of this topic in this chapter, and I’ll leave you with plenty of resources to learn all the details. But you will see how to implement some good, practical security practices here. Then you can build off that based on any regulations your product falls under, such as the [Health Insurance Portability and Accountability Act (HIPAA)](https://www.hhs.gov/hipaa/for-professionals/security/laws-regulations/index.html) for health care apps, [PCI DSS](https://www.controlcase.com/what-are-the-12-requirements-of-pci-dss-compliance/) for fintech apps, or the [General Data Protection Regulation (GDPR)](https://gdpr.eu/checklist/) for almost every app. Let’s start with how users log in to the app with authentication.

# Authentication

_Authentication_ (AuthN) is how the user’s identity is validated by the app. This happens through some method of login. Some of the [common authentication methods](https://www.freecodecamp.org/news/user-authentication-methods-explained/) you’ll work with are:

- Password authentication
    
- Multifactor authentication (MFA)
    
- Token authentication
    
- Biometric authentication
    
- Open authorization (OAuth)
    
- One-time password (OTP)
    

Depending on the app you’re working on, you may use some combination of these methods to give users more login options. You can expect to use at least one or two of these methods on any given app, though.

Ensuring a user is who they claim to be is a complex task. There are many ways a malicious user can try to impersonate a real user. They might try brute force attacks, phishing scams, or hijacking a user’s session token. What you’re trying to do is keep the wrong people out.

The most common method you’ll implement is some form of password authentication. The user’s password will be securely stored in your database, and only their username and password combination will let them log in. Some of the deeper topics behind security will include how [salted, nonrecoverable hashes](https://www.codeproject.com/Articles/704865/Salted-Password-Hashing-Doing-it-Right), [encryption](https://www.okta.com/identity-101/password-encryption/), and decryption work for sensitive user info, such as Social Security numbers or banking information. These topics are out of the scope of this chapter, but I encourage you to do more reading on them to get a better understanding of how and when they are used.

Password authentication can be made even more secure when you add password requirements like a length and specific characters. One of the biggest risks with this authentication method lies with the user. If they have the same password across multiple systems, there’s not much you can do about that with the username-password method alone.

That’s where methods like [MFA](https://aws.amazon.com/what-is/mfa/) and [OTP](https://www.freecodecamp.org/news/how-time-based-one-time-passwords-work-and-why-you-should-use-them-in-your-app-fdd2b9ed43c3/) come in handy because they add another layer of identification on top of the password login. This brings in other tools the user needs access to, such as an email, a phone number, or a special app like [Google Authenticator](https://chrome.google.com/webstore/detail/authenticator/bhghoamapcdpbohphigoooaddinpkbai) or [Okta Verify](https://help.okta.com/en-us/content/topics/mobile/okta-verify-overview.htm). The user typically gets a code they need to put in the app to finish the authentication process. Adding this extra step prevents malicious users from logging in even if they have the password credentials.

I briefly mentioned some of the services you can use for authentication in [Chapter 5](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch05.html#third_party_services), but it’s worth mentioning them again here: [Auth0](https://auth0.com/docs), [FusionAuth](https://fusionauth.io/docs/), [Amazon Cognito](https://docs.aws.amazon.com/cognito/), [SuperTokens](https://supertokens.com/docs/guides), and [Clerk](https://clerk.com/). You also have the option of working with open source options like [Passport.js](https://www.passportjs.org/), [NextAuth.js](https://next-auth.js.org/), and [the built-in functionality in NestJS](https://docs.nestjs.com/security/authentication). It’s usually a good idea to go with one of the third-party services for authentication when you know you need a higher level of security because of industry regulations.

You will also get into details around how you store tokens for a user session and how to handle it when they expire. This will bring up topics like [authorization code flow](https://aaronparecki.com/oauth-2-simplified/#authorization) and [Proof Key for Code Exchange (PKCE)](https://auth0.com/docs/get-started/authentication-and-authorization-flow/authorization-code-flow-with-proof-key-for-code-exchange-pkce). Or maybe you decide it’s better to log the user out and make them log in again after a certain amount of time.

Something to keep in mind as you weigh options is the user experience. Sure, it’s more secure to require biometric authentication, but is it something your users will do? There’s going to be a balance between user comfort and authentication methods that you’ll need to discuss with your team and the Security team.

Once the user has successfully been identified, you need to tell the system the level of access they have. Not all users will have the same functionality level; that’s based on what they need to do in the app.

# Authorization

Even if a user can log in, that doesn’t mean they should have access to everything. So now you have to handle _authorization_ (AuthZ): when you give users a set of permissions they use after authentication. AuthZ is what defines the difference in user roles. For example, a customer support user will have access to many different accounts whereas a customer should be able to access only their account.

Authorization is one of the reasons strong authentication is so important. If a malicious user gains access to one user’s account, that’s bad. If they gain access to an admin user’s account, that’s _really_ bad. Now you have to decide which access control model you’ll work with. A common and very basic thing to implement with authorization is the [principle of least privilege](https://www.cloudflare.com/learning/access-management/principle-of-least-privilege/).

Least-privilege access ensures that users have only the access they really need to perform the tasks for their permission level. For example, someone who manages the docs for an app doesn’t need the same access as someone who onboards users. This should be the base level for any other access control you implement.

You’ll most commonly see [role-based access control (RBAC)](https://www.digitalguardian.com/blog/what-role-based-access-control-rbac-examples-benefits-and-more). RBAC gives you a way to assign users to different roles based on the level of access that role has. Many services you can use for authentication provide some kind of RBAC functionality. You can also build this functionality internally, but keep in mind that you’ll likely end up building frontend features to support internal users updating the roles and users.

Here’s a small example of an RBAC implementation that you can use to build your own functions to check user roles and give access to different parts of the app:

```
export
```

There’s also [attribute-based access control (ABAC)](https://www.okta.com/blog/2020/09/attribute-based-access-control-abac/). ABAC lets you have extremely fine-grained control over what users can do based on their roles, the resources they want to access, the actions they want to take, and the environment they are in. This gives you a more flexible way to implement security because you can keep access very specific based on the exact things a user needs to do. You can expand existing roles to do more or use them as templates for new roles. You’ll find ABAC in action if you ever work with AWS Identity and Access Management (IAM) entities because that’s what they use.

# ANOTHER PERSPECTIVE ON ABAC AND RBAC

Ethan Brown has some good insights into how different types of authorization are implemented. Here are some of his thoughts:

> I think there are different pitfalls in both RBAC and ABAC that can compromise security, and I wouldn’t call ABAC inherently more secure. The danger I see with ABAC is that with such fine-grained access controls, roles can have dozens or hundreds of rules, which makes it harder to reason about and validate specific authorization scenarios. Very often, vast lists of ABAC policies will get copied from one role to another just to save time, but it’s easy to miss policies that were accidentally included but shouldn’t have been.
> 
> RBAC is better in this regard—there are just fewer places for security vulnerabilities to hide in a simpler authorization framework—but RBAC won’t be robust enough for every system. The value of ABAC is that it’s less rigid and allows you to craft new roles and access patterns in the future, ones that you haven’t yet anticipated. If you can envision an application having increasingly complex authorization rules, ABAC in addition to RBAC (I rarely see ABAC without it) may be a better choice.

The biggest drawback to ABAC is that these systems can be complex to set up. Making sure you have the right attributes assigned to the correct roles is the most important part. It’s tedious to initialize, but over time the payoff can be worth it if your application has complex permission requirements. You’ll be able to copy-paste a lot of things for new attributes as roles change over time. Be careful with copying roles when you use ABAC because you could accidentally give a role more permissions than it needs.

Here’s a small example of an ABAC definition:

```
export
```

The last model I’ll cover is [policy-based access control (PBAC)](https://www.nextlabs.com/what-is-policy-based-access-control/), which is very similar to ABAC. Both are referred to as _fine-grained access control (FGAC)_. The main difference between the two is that PBAC uses the policies you define compared to the attributes you define in ABAC. It’s just a different way of defining how you want to restrict access. This is another access model that your cloud provider might have available.

Many times, access control is managed by the Security team or a user support team once the dev and Product teams have documented how different users should be able to interact with the app. Other times, you will be able to directly add new attributes or roles and change existing permissions. A good rule to follow is granting all users the least privilege they need to get started and then adding more permissions as they need them.

All of these access models have trade-offs. RBAC is probably the fastest to implement, but managing all the roles over time can get messy. If you foresee the app growing to a level where RBAC will get unwieldy, then it’s time to consider adding ABAC or PBAC. These models will take more time to set up, but they will be easier to manage over time. When you have to decide which model to use, here are a few questions you can ask:

- What are the different types of users and the access level for each type?
    
- Do any of the services you use offer any access control functionality?
    
- What level of granularity do you need when it comes to access?
    
- How flexible or dynamic do access levels need to be?
    
- What information do you have available to determine a user’s access level?
    

This barely scratches the surface of how deep you can go into authorization. I encourage you to check out some of the resources in this section. These are some different strategies you can choose from; you’ll work with the Security and Support teams to fully implement them across the company.

With this background on AuthN and AuthZ, you can move on to some of the most common security risks online and how your work helps prevent a lot of them.

# OWASP Top 10

Regardless of whether you’re familiar with the [OWASP Top Ten](https://owasp.org/www-project-top-ten/), you should look at all of them in detail to understand the security side of your code. A few of these in particular fall under what we’ve covered in this chapter. Broken access control is the number-one vulnerability on the list, so you know it’s an area worth spending time on.

[Broken access control](https://owasp.org/Top10/A01_2021-Broken_Access_Control/) is something that can easily slip through the cracks as roles and users change. One of the most common ways this becomes an issue is not following the principle of least privilege. It can get to the point where you or someone on the Security team is constantly having to update permissions, so eventually they give everyone the same permissions to make it easier. This can lead to people who should only be able to read data also having the ability to write data, whether they know it or not.

Another really common way that access control can be bypassed is by not having access control on certain resources. You might remember to put access control guards on POST endpoints, but maybe you leave them off PUT or DELETE endpoints. This is a way users can work around the permissions you think you have in place. One way to avoid this is to deny access by default. That way, you have to explicitly allow users to have access to resources.

Another Top 10 issue is [injection](https://owasp.org/Top10/A03_2021-Injection/) due to not validating inputs. You need to validate all the inputs you receive from the frontend or direct endpoint calls. Some devs make the mistake of thinking frontend validation will cover them, and that’s incorrect. Anytime an endpoint receives a request where the user passes parameters, those values should be validated and sanitized to prevent things like SQL injection attacks and cross-site scripting attacks.

[Authentication failure](https://owasp.org/Top10/A07_2021-Identification_and_Authentication_Failures/), another top 10 security issue, happens when you have any weaknesses in your authentication process, such as allowing weak passwords, putting credentials directly in the URL, or even having weak reset-password flows. Some of these weaknesses come from wanting to cater to user comfort. This is part of those trade-offs you have to consider when you decide what type of authentication to use.

Some ways you can avoid authentication failure are by enforcing strong user passwords, adding MFA, and limiting the number of logins a user can attempt. These aren’t foolproof, and there are still ways for a determined attacker to try to manipulate your authentication. But when you have these things in place along with all the other security layers involved, you have a higher likelihood of preventing attacks.

One vulnerability that often gets overlooked is [vulnerable and outdated components](https://owasp.org/Top10/A06_2021-Vulnerable_and_Outdated_Components/). Once security vulnerabilities have been discovered and patches have been released, any outdated versions of packages or tools will have well-documented attack vectors. Keeping your packages up to date is just as important for security as it is for code maintenance. Making sure your infrastructure is using the most up-to-date version of services is another way to keep everything as secure as possible.

The last vulnerability from the top 10 that I’ll go over is [insecure design](https://owasp.org/Top10/A04_2021-Insecure_Design/) because it’s an interesting one. Sometimes your app can be insecure by design or a lack of design. You should consider doing some [threat modeling](https://cheatsheetseries.owasp.org/cheatsheets/Threat_Modeling_Cheat_Sheet.html) to address this. The basic process includes four questions:

- What is the feature we are working on?
    
- What can go wrong with it?
    
- How are we going to handle it?
    
- Did we do a good enough job addressing the concerns?
    

A scenario where this can happen is when placing orders. Is there a way for users to bypass the number of items you actually have in stock? Can they change any order details from an email link? Is there a chance that users might see the wrong information based on conditional checks you have in the code?

It can be harder to spot these vulnerabilities because sometimes they’re part of the feature requirement. This is where you have to look at a feature from every angle you can think of. If you need to bring in other teams to get an idea of how something would work, set up a call. Point out any security issues you see when a product owner is explaining how they’d like a feature to work. It might lead to a completely different path for a feature.

Make sure you take a look at the remaining vulnerabilities in the top 10. Some of these will be discussed later in the book when we get to [Chapter 19](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch19.html#frontend_security_considerations) on frontend security. But there are a few general security practices you can add to your toolbox that don’t fall under a specific topic.

# Other Noteworthy Practices

Regardless of the industry regulations your product falls under, you should always implement some kind of audit trail for any data changes or event triggers. At the bare minimum, include fields like the updated date and the ID of the user who made the update. This can help you identify users and services that are causing issues with things like unexpected data access, and you can determine when the attack started. You might even consider adding audit trails on your GET requests if you want to track who accessed data across time.

Using a tool like [Snyk](https://snyk.io/) or [Veracode](https://www.veracode.com/) is a way to automatically scan your code for most known security vulnerabilities. You’ll include these kinds of tools in your DevOps pipelines; that will come up in [Chapter 27](https://learning.oreilly.com/library/view/full-stack-javascript/9781098122249/ch27.html#building_a_cisoliduscd_pipeline). These tools will even point out dependencies that have vulnerabilities in them. When you get reports on the app’s vulnerabilities, take some time to go through them and think about the best approach to handling them. You may decide to replace packages or use other services.

It’s important to note that there are a few different types of security testing going on. One of the first you should implement is [Static Application Security Testing (SAST)](https://www.opentext.com/what-is/sast). This will analyze your code for known vulnerabilities and insecure code patterns. This is how you can check your code before you run it. To check your code while it’s running, similar to how an attacker would, you can use [Dynamic Application Security Testing (DAST)](https://www.opentext.com/what-is/dast). You’ll work with the DevOps team to add this to your pipeline.

Advocate for things like regular [penetration testing (pen testing)](https://www.cloudflare.com/learning/security/glossary/what-is-penetration-testing/). This is usually when a team of ethical hackers are contracted to try to find weaknesses in your system. They write up reports with their findings and some potential solutions. This might also fall under the tasks that the Security team handles. Some companies even put out [bug bounties](https://hackerone.com/bug-bounty-programs) for anyone who can find legit vulnerabilities in their live apps. Apps need to be attacked from as many angles as possible so that you can proactively harden them against real attacks.

Real attacks usually come from [black-hat attackers](https://www.kaspersky.com/resource-center/threats/black-hat-hacker). They are the opposite of white-hat attackers, who only use their skills to help identify and resolve vulnerabilities. Black-hat attackers are who you’re trying to keep from getting unauthorized access to different parts of your app. There are also [grey-hat attackers](https://www.devry.edu/blog/grey-hat-hacker.html), who usually find vulnerabilities without permission but don’t exploit them with malicious intent.

Do regular industry-specific audits to make sure the app still meets all the required regulations. Likely, a legal team will be involved somewhere in the background to make sure everyone remembers this, but if not, go through your code and make sure it’s compliant. Encrypting user PII and giving users access to their data are examples of things you should check for.

Update your dependencies as often as you can so that you don’t miss any security patches. A tool like [Dependabot](https://docs.github.com/en/code-security/getting-started/dependabot-quickstart-guide) will help you with this. You’ll find that some apps are several major versions behind on certain packages, even though the vulnerabilities are well known and documented. This is one of those tech-debt things you’ll want to work with Product to prioritize in the backlog. It’s much easier to do updates incrementally than trying to update when there have been several breaking changes.

Something else you need to consider with your dependencies is supply chain security. We often use open source software, which has its own dependencies that could have vulnerabilities. These vulnerabilities could be intentional, such as if a package causes [distributed denial of service (DDos) attacks on your server](https://snyk.io/blog/open-source-npm-packages-colors-faker/). Or they could be accidental, such as if someone on the team installs a [copycat version of a package](https://www.bleepingcomputer.com/news/security/copycats-imitate-novel-supply-chain-attack-that-hit-tech-giants/). You can use this [supply chain security framework](https://github.com/ossf/s2c2f/blob/main/specification/framework.md) to help watch out for any vulnerabilities introduced from third-party dependencies in your app.

You also need to rotate any access tokens or credentials you use with your third-party services or other tools. I’ve seen companies that have had the same tokens for years and then had to take days to figure out why the system is broken because the service they used finally expired the token. Rotate them a few times a year at least, but see if the Security team has any rules.

# CONSIDERING DEVELOPER EXPERIENCE WITH AUTHZ TESTING

Here are more great insights about working on real-world projects from Ethan Brown:

> The DX of AuthZ is something that often gets overlooked. I worked on a large project that had extensive AuthZ requirements. At first, all the developers just gave themselves a “superuser”-type policy so they could get features done. The problem is, when we started doing user acceptance testing with actual users with restricted permissions, a lot of functionality was broken because developers hadn’t “role-played” as they were developing; they just did everything as a superuser. This led us to introduce a “Developer” policy, which didn’t directly provide any elevated authorization except the ability to assume any role with a handy “role switcher.”
> 
> This got development moving more smoothly. Once devs could easily switch roles without logging out and in or having multiple user accounts or multiple browsers, role-based feature development got a lot easier. Not only does it make development less painful, it subtly keeps roles in the front of developer’s minds, so they’re more aware of AuthZ considerations.

Take some time to learn how to do some basic attacks yourself! This falls under ethical or [white-hat hacking](https://www.hackerone.com/knowledge-center/white-hat-hacker), where you can use hacking skills to find and document vulnerabilities so that they can be remedied. [PortSwigger’s Web Security Academy](https://portswigger.net/web-security/all-topics) has great resources on the different types of attacks that commonly come up. You can also look up some of the tools attackers commonly use, such as [sqlmap](https://sqlmap.org/), [Wireshark](https://www.wireshark.org/), [John the Ripper](https://github.com/openwall/john), or even the whole [Kali Linux OS](https://www.kali.org/). You don’t need to be an expert, but it’s cool to see how things work from an attacker’s perspective. It’ll help you learn ways to better defend your code because you know what attackers will be trying to do.

# Conclusion

This chapter was more of a discussion on security than going over any code implementation. That’s because the things you need to look out for can be less obvious than the code you’ll write. Security concerns slip into places you may not originally look at, such as logging and monitoring or feature requirements. We’ll go over logging in detail in the next chapter because it also helps with debugging.

The goal of this chapter is to make you aware that you should consider security at every point of development, from discussions with other teams to the code you write. There are already a lot of questions to ask as you develop, but one that should stay at the top of your mind is this: is this secure? If you can show a vulnerability in it, then someone whose specialty is security can probably find even more. Your knowledge of security can go as deep as you like, but you don’t need to know the details of hashing math and encryption to understand how to effectively use the tools.