# DevOps for Horses: Moving an Enterprise Application to the Cloud



Written by: Eric Shamow Edited by: Michelle Carroll As an engineer, when you first start thinking about on-demand provisioning, CD, containers, or any of the myriad techniques and technologies floating across the headlines, there is a point when you realize with a cold sweat that this is going to be a bigger job than you thought. As you watch folks talking at various conferences about the way they are deploying and scaling applications, you realize that /your applications won’t work/ if you deployed them this way. Most of the glamorous or really interesting, thought-provoking discussions around deployment methodologies work because the corresponding applications were built to be deployed into those environments in a true virtuous cycle between development and operations teams. Sometimes the lines between those teams disappear entirely. In some cases, this is because Operations is outsourced entirely — consider PaaS environments like Heroku or Google App Engine, where applications can be deployed with tremendous ease, due to a very restricted set of conditions defining how code is structured and what features are available. Similarly, on-premises PaaS infrastructures, such as Cloud Foundry or OpenShift, allow for organizations to create a more flexible and customized environment while leveraging the same kind of automation and tight controls around application delivery. If you can leverage these tools, you should. I advise teams to try and build out an internal PaaS capability — whether they are using Cloud Foundry or bootstrapping their own, or even several to allow for multiple application patterns. The Twelve-Factor App pattern is a good checklist of conditions to start with for understanding what’s necessary to get to a Heroku-like level of automation. If your app meets all these conditions, congratulations — you are probably ready to go PaaS. My App Isn’t Ready For PaaS Unless you’re a startup or have a well-funded team effort to move, your application won’t work as it stands in a PaaS. You are perhaps ready for IaaS \(or are evaluating IaaS\) wondering, where do I start? If you can’t do much with the application design, how can you begin to get ready for a cloud move with the legacy infrastructure and code you have? Getting Your Bearings Start by collecting data. A few critical pieces of information I like to gather before drawing up a strategy:

* What are the components of the application? Can you draw a graph of their dependencies? 
* If the components are separated from one another, can they tolerate the partition or does the app crash or freeze? Are any components a single point of failure? 
* How long does it take for the application to recover from a failure? 
* Can the application recover from a typical failure automatically? If not what manual intervention is involved? 
* How is the application deployed? If the server on which the application is running dies, what is the process/procedure for bringing it back to life? 
* Can you easily replicate the state of your app in any environment? Are your developers looking at code in an environment that looks as close as possible to production? Can your QA team adequately simulate the conditions of an outage when testing a new release? 
* How do you scale the application? Can you add additional worker systems and scale the system horizontally, or do you need to move the system to bigger and more powerful servers as the service grows? 
* What does the Development/QA cycle look like? Is Operations involved in deploying applications into QA? How long does it take for developers to get a new release into and through the testing cycle? 
* How does operations take delivery of code from development? What is the definition of a deliverable? Is it consistent, or does it change from version to version? 
* How do you know that your application was successfully installed? 

  I’m not going to tackle all of them, but will rather focus on some of the key themes we’re looking for in examining our apps and environment. 

  Coupling

  One of the key underpinnings of modern application design is the understanding that failure is inevitable — it’s not a question of if a component of your application will fail, but when. The critical metric for an application is not necessarily how often it fails \(although an app that fails regularly is clearly a problem\) but how well its components tolerate the failure of other components. As your app scales out — and particularly if you are planning to move to public cloud — you can expect that data will no longer flow evenly between components. This is not just a problem of high latency, but variable latency — sudden network congestion can cause traffic between components to be bursty.

  If one component of your application depends on another component to be functional, or your app requires synchronous and low-latency communication at all times between components, you have identified tight coupling. These tight couplings are death for applications in the cloud \(and they’re the services that make upgrades and migration to new locations the most difficult as well\). Tight couplings are amongst the most difficult problems to address — often they relate to application design and are tightly tied to the business logic and implementation of the application. A good overview of the problem and some potential remedies can be found in Martin Fowler’s 2001 article “Reducing Coupling” \(warning: PDF\).

  For now , we need to identify these tight couplings and pay extra attention to them — monitor heavily around communications, add checks to ensure that data is flowing smoothly, and in general treat these parts of our architecture as the fragile breakpoints that they are. If you cannot work around or eliminate these couplings, you may be able to automate processes for detection and remediation. Ultimately, the couplings between your apps will determine your pattern for upgrades, migrations and scaling — so understanding how your components communicate and which depend on each other is essential to building a working and automated process.

  Automation

  If you can’t reinstall the app without human intervention, you have a problem. We can expect that a server will eventually fail and that application updates will happen on a regular cadence. Humans screw up things we do repetitively — repeat even a simple process often enough and you will eventually do it wrong. Computers are exceptionally good at repetitive tasks. If you have your sysadmins doing regular installs of your applications — or worse if your sysadmins have to call in developers and they must pair to slowly work through every install — you are not taking advantage of the computers. And you’re overtaxing humans who are much better at — and happier — doing other things.

  Many organizations maintain either an installation wiki, a set of install scripts, or both. These sources of information frequently vary and operators need to hop from one to the other to assemble and install. With this type of ad-hoc assembly of a process, it’s likely that one administrator will not follow the process perfectly each time, but certain that different administrators will follow the process in different ways. Asking people to “fix the wiki” will not fix the discrepancy. The wiki will always lag the current state of your systems. Instead, treat your installation scripts like “executable documentation.” They should be the single source of truth for the process used to deploy the app.

  While you will want your automation to use good, known frameworks, the reality is that a BASH script is a good start if you have nothing in place. Is BASH the way to go for your system automation? As a former employer put it, “SSH in a for loop isn’t enough” — and it’s not. But writing a script to deploy a system in a language you already know is a good way to identify if you /can/ automate the deploy, as well as the decisions you need to make during the install. This information informs your later choice of automation framework, and enables you to identify which parts of your configuration change from install to install. As a bonus, you’ve taken a first pass at automating your process, which will speed up your deploys and help you select an automation framework that best fits your use case. For an exploration of this topic and an introduction to taking it a step further into early Configuration Management, check out my former colleague Mike Stahnke’s dead-on 2013 presentation “Getting Started With Puppet.” 

  Environment Parity and Configuration Management

  We’ve all been on some side of the environment parity issue. Code makes it into production that didn’t take into account some critical element of the production environment — a firewall, different networking configuration, different system version, and so on. The invariable response from Operations is, “Developers don’t understand real operating environments.” The colloquial version of this is, “It works on my laptop!”

  The more common truth is that Operations didn’t provide Development with an environment that looked anything like production, or even with the tools to know about or understand what the production environment looks like. As an Operations team, if you don’t offer Development a prod-like environment to deploy into and test with, you cede your right to complain about code they produce that doesn’t match prod.

  Since it is often not possible to give developers an exact copy of production, it’s important for the Operations team to abstract away as many changes between environments as is possible. Dev, Prod, QA and all other teams should be running the same OS versions and patch sets, with the same dependencies and same system configuration across the board. The most sensible way to do this is with Configuration Management. Configure all of your environments using the same tools and — most critically — with the same configuration management scripts. The differences between your environments should be a set of variables that inform that code.

  If you can’t reduce the differences between your environments to code informed by variables, you’ve identified some hard problems your developers and operations teams are going to have to bridge together. At the very least, if you can make your environments /more/ similar, you can significantly reduce the number of factors that must be taken into account when an app fails in one environment when it succeeded in another.

  Get Operations out of the Dev/QA Cycle

  The notion of Operations being required to install applications into a QA/Testing environment always baffled me. I was in favor of Development not doing the install themselves, but I also understood that opening a ticket with Operations and waiting for an install is a time-intensive process, and that debugging/troubleshooting is a highly interactive one. These two needs are at odds. By slowing down the Dev/QA feedback loop, Operations not only causes Development to become less efficient, it also encourages developers to do larger chunks of work and submit them for testing less frequently.

  The flip side of this is that allowing developers full root access on QA servers is potentially dangerous. Developers may inadvertently make changes that change the performance of the servers from production. Similarly, if developers are installing directly into QA, operations doesn’t get to look at the deployable until it reaches production. When they install the application for the first time, it’s in the most critical environment.

  There’s a three-part fix for this:

* Developers are responsible for deliverables in a consistent format. Whether that’s a package, a tarball, or a tagged git checkout, the deliverable must look the same from release to release 
* QA is managed via Configuration Management, and applications are installed into QA using the same automation tools/scripts used in production. 
* Operations’ SLA for QA is that it will flatten and re-provision the environment when needed. If a deployment screws up the server, Ops will provide a new, clean server. 

  Using these policies, the application is installed into QA and any subsequent environments with the same scripts. If we’ve learned anything from the Lean movement, it’s that accuracy can be improved by reducing batch sizes, increasing the speed of processing and baking QA into the process. With these changes, the deployment scripts and artifacts are tested dozens, hundreds or thousands of times before they are ever used in production. This can help find deployment problems and iron out scripts long before code ever reaches user-facing systems.

  The benefits for both teams are clear: Development gets a fast turnaround time for QA, Operations gets a clean deliverable that can be deployed via its own scripts.

  Functional Testing

  While there will always be the need for manual testing of certain functionality, establishing an automated testing regimen can provide quick feedback about whether an app is functioning as intended.

  While an overview of testing strategies is beyond the scope of this article \(Chapter 4 of Jez Humble and David Farley’s book Continuous Delivery provides an excellent overview\), I’d argue for prioritizing a combination of functional and integration tests. You want to confirm that the app does what is intended. Simple smoke tests to verify that a server is configured properly and that an application is installed and running is a good first pass at a testing regimen.

  Once you get comfortable writing tests, you should begin doing more involved testing of application and server behavior and performance. Every time you make a change that alters the behavior of the application or underlying system, add a test. Down the line you may want to consider TDD or BDD, but start small — having imperfect tests is better than having no tests at all.

  At the application level, your development team likely has a testing language or suite for unit and integration tests. There are a number of frameworks you can use for doing this at the server/Configuration Management level. I have used both serverspec and Beaker with success in the past.

  The first time you run a proposed configuration management change through tests and discover that it would break your application is a revelation. Similarly, the first time you prevent a regression by adding a check for something that “always” breaks will be the last time somebody accidentally breaks it.

  Wrapping Up

  We’ve just scratched the surface of what can be done with an existing environment, but as you can hopefully see, there’s plenty you can do right now to get your environment ready for IaaS \(and eventually, PaaS\) without touching your application’s code.

  Remember that this process should be iterative — unless you have the budget to build a greenfield environment tomorrow, you are going to be tackling this one piece at a time. Don’t feel ashamed because your environments aren’t automated /enough/ or you don’t have comprehensive enough tests for your application. Rather, focus on making things better. If you don’t have enough automation, build more. If there aren’t enough good tests, write just one. Then re-examine your environment, see what most needs improvement, and iterate there.

  There’s no way to completely move an app without touching the code, but there’s plenty of work to do before you get there in preparation of scalable, loosely coupled code. Don’t wait for the perfect application to start doing the right thing.

  Posted by Christopher Webber 

