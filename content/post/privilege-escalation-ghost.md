+++
author = ["david-lindner"]
title = "Privilege Escalation in Popular Blogging Platform"
date = "2019-04-29"
description = "Matt Austin discovers a privilege escalation in Ghost."
tags = [
    "privilege-escalation",
    "xss",
    "threat-hunting",
    "poc",
]
thumbnail = "images/privesc.png"

+++

[Ghost](https://github.com/TryGhost/Ghost) is a popular open source blogging platform written in Node.js. It is downloaded around 8,500 times a week according to [npm](https://www.npmjs.com/package/ghost).

Like all good open source projects, Ghost has a defined process for [reporting vulnerabilities](https://ghost.org/docs/security/#reporting-vulnerabilities). On July 18, 2018, Contrast Security's Matt Austin discovered Ghost users could escalate their privileges by embedding malicious JavaScript in their blog entries. His submission was sent to [security@ghost.org](mailto:security@ghost.org) as outlined in Ghost’s responsible disclosure guidelines.

## Matt's Submission

### Description

A non-administrative user (editor, author, contributor) can create a post with a “Code Injection” into the blog header. This user can add javascript to the header that when viewed by an administrator is executed as the administrator.

### Steps to Reproduce

1. Login as an underprivileged user. (named Bob in the example below)
2. Create a new post or story.
3. Click on the Gear icon on the top right and click “Code Injection”
4. Add the following code to the “Post Header” in a `<script>` tag: https://gist.github.com/matt-/15285acc7365997eb6ae6af914fb3105#file-make_admin-html  
(Change the name “Bob” in the code to your current user name)
5. If you press save (cmd+s on mac) and click on the View Preview button:
6. The link will look something like http://someblog.com/p/48ba590b-7e25-494f-b9e8-d9a6d7e5eb1b/
7. Send this link to an Administrator user (hide it in an iframe, mask it with a URL shortener). When viewed the injected code will modify the user (“Bob” in this case) giving him the Administrator role.

### POC Video

{{< youtube 4zGrdDusEjo >}}

As you can see in the video, Matt was able to successfully elevate his privilege from an “Author” to an “Administrator” simply by getting an Administrator user to click on the blog preview link.

Based on Ghost’s documentation on user roles, there are 5 distinct roles used for different privilege within the Ghost system.

* Contributors: Can log in and write posts, but cannot publish
* Authors: Can create and publish new posts and tags
* Editors: Can invite, manage and edit authors and contributors
* Administrators: Have full permissions to edit all data and settings
* Owner: An admin who cannot be deleted and has access to billing details

However, Matt never received any feedback from his submission to Ghost about the privilege escalation bypass. On October 16, 2018, a new [commit](https://github.com/TryGhost/docs/commit/c65058852181813a216fdc8b8134dc7c4e1aba84#diff-cb5c147b83c6023fea4cec2b394f3fd4) was submitted to the GitHub repository for Ghost that said:

```
#### However, we're generally _not_ interested in...

- [Privilege escalation](#xss--privilege-escalation-attacks) as result of trusted users publishing arbitrary JavaScript<sup><a href="#xss--privilege-escalation-attacks">1</a><sup>
```

Still, no response from Ghost to Matt’s submission, not even a thank you for your submission, we have updated our disclosure program to not involve privilege escalation.

Ghost also added the following, explaining their position:

```
Privilege escalation attacks

Ghost is a content management system and all users are considered to be privileged/trusted. A user can only obtain an account and start creating content after they have been invited by the site owner or similar administrator-level user.

A basic feature of Ghost as a CMS is to allow content creators to make use of scripts, SVGs, or embedded content that is required for the content to display as intended. Because of this, there will always be the possibility of "XSS" attacks, albeit only from users that have been trusted to build the site's content.

Ghost's admin application does a lot to ensure that unknown scripts are not run within the admin application itself, however, that only protects one side of a Ghost site. If the front-end (the rendered site that anonymous visitors see) shares the same domain as the admin application then browsers do not offer sufficient protection to prevent successful XSS attacks by trusted users.

If you are concerned that trusted users you invite to create your site will act maliciously the best advice is to split your front-end and admin area onto different domains (e.g. https://mysite.com and https://mysiteadmin.com/ghost/). This way browsers offer greater built-in protection because credentials cannot be read across domains. Even in this case, it should be understood that you are giving invited users completely free reign in content creation so absolute security guarantees do not exist.

We take any attack vector where an untrusted user is able to inject malicious content very seriously and welcome any and all reports.
```

As of April 29, 2019, Matt had not received a response from Ghost, so we are following our normal process of disclosure, as we feel the issue at hand warrants a fix, or at least a discussion from the community. We want applications to be secure, so let’s work together to fix the issues that need to be fixed and make sure all users feel safe using the Ghost platform.