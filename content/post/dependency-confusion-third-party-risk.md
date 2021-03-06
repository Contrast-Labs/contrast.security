+++
author = ["matt-austin"]
title = "Dependency Confusion: A New Third-Party Risk for the Software Factory"
date = "2021-02-24"
description = "Matt Austin reviews the risk of dependency confusion."
tags = [
    "thought-leaders",
    "hacked",
    "threat-hunting",
    "dependency-confusion",
]
thumbnail = "images/dependencies.png"

+++

The SolarWinds attack has been extensively covered over the past two months—and rightly so. It has been characterized as among the worst hacks of the past 10 years, targeting SolarWinds’ software factory and compromising the code in software updates delivered to its customers. 

Russian nation-state attackers first gained access to the server development environment for the SolarWinds Orion infrastructure management platform in [September 2019](https://www.itpro.com/security/358288/solarwinds-hackers-breached-systems-september-2019) — 14 months before the attack was discovered. Microsoft President Brad Smith estimates that more than [1,000 Russian engineers](https://www.infosecurity-magazine.com/news/microsoft-1000-hackers-worked/) worked on the project.

After moving laterally in the SolarWinds network, the attackers used [SUNSPOT](https://www.crowdstrike.com/blog/sunspot-malware-technical-analysis/) malware to insert a dynamic link library (called [SUNBURST](https://www.cynet.com/attack-techniques-hands-on/sunburst-backdoor-c2-communication-protocol/)) to create a backdoor into more than [18,000 networks](https://www.bloomberg.com/news/articles/2020-12-18/russia-linked-solarwinds-hack-ensnares-widening-list-of-victims) belonging to customers that installed the malicious update. These customers include [multiple U.S. federal government agencies](https://www.cnn.com/2020/12/16/politics/us-government-agencies-hack-uncertainty/index.html) and major [private corporations](https://www.businessinsider.com/solarwinds-hack-explained-government-agencies-cyber-security-2020-12) like Microsoft, Cisco, Intel, and Deloitte.

## Protecting the Entire Software Supply Chain

The SolarWinds event is a wake-up call to all organizations to prioritize application security—not just because of massive attacks like this one, but also because of the smaller-scale attacks that occur every day—and can potentially bring significant damage to organizations of all sizes.

This event is also a reminder that organizations need to protect every element of the software supply chain, including:

* _What you write_: Custom code developed in-house
* _What you build with_: Hundreds of software development tools in use at many organizations
* _What you buy_: Off-the-shelf Software-as-a-Service (SaaS) applications
* _What you use_: The numerous third-party libraries that most applications depend on


Organizations must proactively work to prevent and remediate vulnerabilities in every part of the software supply chain. Recent research uncovered a new vulnerability for organizations that utilize internal and third-party libraries in their applications. But before we talk about this newly discovered dependency confusion vulnerability, I want to discuss a long-time attack tactic that is an antecedent to dependency confusion.

## Typosquatting: Taking Advantage of Mistyping and Inattention

The SolarWinds attack was far from the first in which the source code of an application was manipulated by bad actors. One example is [typosquatting](https://www.csoonline.com/article/3600594/what-is-typosquatting-a-simple-but-effective-attack-technique.html), which cyber criminals have used for numerous years dupe developers into using contaminated open-source libraries. The principle is simple: Busy developers’ fingers sometimes hit an incorrect key on the keyboard, so attackers reserve package names that are very similar to a legitimate package name to mimic the library that is being sought. 

Consider the following example. If the name of the legitimate package is Express, a cyber criminal might reserve “exprss,” “expres,” and “xpress” in the hope that developers will mistype the name. If the misspelled name returns an existing package, the developer might simply click on it rather than noticing that it is not the intended package. 

To avoid tipping off the developer of the error, cyber criminals program the malicious package to behave like the legitimate one — nothing seems out of the ordinary. But after the package is in the code base, the attacker sends out an update to the package to deliver malicious content into the software.

To be successful, typosquatting requires victims to mistype the name of a package — and not notice the typo. And while some package managers try to combat the tactic by disallowing common misspellings of some words as package names, there is no way to eliminate every conceivable typo from the universe of available package names. 

## Dependency Confusion: Not Dependent on Misspelling

Dependency confusion is a newly discovered vulnerability that attackers can use in a manner similar to typosquatting. It involves creating a package on an external library with the same name as an internal library in use at a particular organization. When they are not configured effectively, package managers often routinely default to external libraries over internal ones, while others access external libraries when an internal one is unavailable, such as when a user forgets to activate the VPN. These attacks can be automated to multiply their impact.

When an internal library and an external library have the same name, hurried developers do not even need to mistype the package name for the attack to penetrate the continuous integration/continuous deployment (CI/CD) environment — and potentially slip into production. Developers use “internal” modules that are not scoped, and the module is available for an attacker to potentially register.

## Research Uncovers Dependency Confusion Risk

Security researcher Alex Birsan recently [used the tactic](https://blog.sonatype.com/sonatype-spots-150-malicious-npm-packages-copying-recent-software-supply-chain-attacks) to push malicious proof-of-concept (POC) code to internal development builds at more than 35 technology companies. When activated, his package simply returned a message that the package is “for security research purposes only.”      

One difficulty that cyber criminals—and security researchers—have in deploying a dependency confusion attack is that they must figure out the names of an organization’s internal libraries and modules. But Birsan was able to find documents that list the names of internal packages at many companies. While organizations may not have considered the lists themselves to be sensitive information, they provided a key through which an attacker (fortunately, an ethical one in this instance) was able to infiltrate the software factory at some of the world’s most technologically advanced companies.

After Birsan’s research was reported, hundreds of copycats published similar Node Package Manager (npm) packages—ostensibly for research purposes. In addition, at least [one apparent malicious attack](https://info.qentinel.com/blog/dependency-confusion-attack) using dependency confusion has been reported. Qentinel revealed that an unknown account created libraries with names identical to the names of internal libraries on the PyPI open-source repository for Python. Fortunately, bad actors had not yet populated them with source code. The company’s pip package manager defaults to PyPI, resulting in the dependencies failing to build.

## Managing the Risks of Package Managers

Package managers are one of many software tools that help developers work more efficiently, but this apparent attack illustrates that organizations must be cognizant of the risks they pose. As libraries and the software in which they are used become increasingly complex, organizations become more vulnerable to risks like dependency confusion. 

One area of great concern with package managers is post-install scripts. When a library is downloaded from the package manager, the first thing the library does is run the post-install script. This is one method by which a malicious payload can be delivered in a dependency confusion attack. In npm, for example, these scripts do not even need to match the application programming interface (API) of the internal dependency. Organizations must pay more attention to the security of these tools going forward.

## Protecting Against Dependency Confusion Vulnerabilities

There are several things that developers can do to protect against dependency confusion:

* _Do a better job of tracking dependencies._ Many organizations have limited visibility into the sources of their software. They need to understand these dependencies and track them at a granular level.
* _Treat lists of internal libraries and packages as sensitive information._ Since the success of an attack depends on knowing the name of an internal resource, organizations can no longer be casual about where this information is stored and shared.
* _Activate namespaced modules._ Many package managers, including Maven and npm, now support namespaced modules, which prevents the same name from being used for two different resources. 
* _Register namespaces on public repositories._ Another strategy is to register the names—and similar spellings—of internal libraries and packages on all public repositories being used by the organization—even if the reserved names are never used. This would prevent attackers from tricking a developer into using an identically or similarly named package on a public repository.

## Enhanced Features in Contrast Application Security Platform

In response to this new research, Contrast Labs and Contrast engineers embarked on a whirlwind development effort the day after Birsan’s research was published to help organizations deal with this new threat. The result is several new features in the Contrast Application Security Platform, including the command-line interface the (CLI). The Contrast CLI is available to all customers using the latest version of the Contrast Application Security Platform. These include:      

* Developer enablement in Contrast CLI
  * Flag any internal libraries included in the package.json as a risk for dependency confusion
  * Highlight libraries not scoped for the project as a warning
  * Highlight release history to indicate if an open-source project is still being maintained before merging
* Security benchmarking in the Contrast Application Security Platform
  * Updates for alerts on libraries with suspicious versioning (coming soon)  

## Stay Tuned For More!

Keep your eyes open for a follow-up blog describing some of my own discoveries about dependency confusion. I am actually one of the “copycats” researching these vulnerabilities since the news of Birsan’s work has been revealed and used this method to identify a dependency confusion issue with a software tool used by millions of workers.

Additional resources readers may want to check out include the upcoming webinar “[How Dependency Confusion Poses a Serious Risk in the Software Supply Chain](https://www.contrastsecurity.com/webinar/how-dependency-confusion-poses-a-serious-risk-in-the-software-supply-chain?utm_campaign=WB-swsupchain-N-Q1FY22&utm_medium=email&_hsmi=112430204&_hsenc=p2ANqtz-_oISSnTOAus6qoDHHiJDFo3upN2n8n08Xax2azagfhrYtrTGe3JJKgr7Gx6KiOj1yVUt8ZrHqyPk2-ZLYBqwdk6W0MR38pxajcGmiHfg7mePRBnRs&utm_content=112430204&utm_source=hs_email&hsLang=en-us)”, and my podcast interview, “[New Open-Source Dependency Confusion Vulnerability Threatens Software Supply Chain](https://soundcloud.com/contrastsecurity/new-open-source-dependency-confusion-vulnerability-threatens-software-supply-chain)”.      