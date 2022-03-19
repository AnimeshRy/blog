---
layout: post
title: Keycloak - Should you do your own authentication?
subtitle: Simple Guide to Keycloak
categories: Authentication & Authorization
tags: [authentication, web-applications, authorization, self-hosted]
---

Authentication is a major part of any web application where there can be multiple mechanisms put to be implemented, protocol to follow, and even then there can be gaps that lead to data breaches.

Last month, we ended up having a work problem where we wanted a robust and reliable solution that could implement multiple authentications flows for us. We gave a thought about building one ourselves but due to time constraints, we explored some of the existing solutions out there.

There were a number of parameters on which we determined a solution out there -

- Pricing
- Open Source
- Ease of Use
- Scalable
- Self Hosted

There are a bunch of solutions on the market — both free and paid — that promise to provide such features. In today’s article, I will try to present you one of these tools, which as you probably guess from the title, will be Keycloak.

Upon researching a bit, Okta and Auth0 came out as “the pricey contenders” but on the other hand, Keycloak offered a totally free self-hosted solution that fit our use case pretty well. Although the ease of use was a constraint in this case still Keycloak was the one we decided on.

## So what really is Keycloak?

It is a [tool](https://www.keycloak.org/) for “Identity and Access Management”, as written on their project [page on GitHub](https://github.com/keycloak/keycloak). Additionally, Keycloak is an open-source tool currently licensed with Apache License 2.0. It is also an upstream project for Red Hat SSO, so if you are looking for something more enterprise-centered, you can check it.

The full list of supported platforms depends on which protocol you decide to use, currently, Keycloak supports three different protocols, and can be viewed in the [documentation](https://www.keycloak.org/docs/latest/). Keycloak’s initial release took place in September 2014. It is developed and maintained by people from Red Hat. They are open to new contributors if anyone is interested.

I think it is time to tell you more about what Keycloak can do.

## What does Keycloak really offer?

- **Multiple Protocols Support -** As for now, Keycloak supports three different protocols, namely - OpenID Connect, OAuth 2.0, and SAML 2.0.
- **SSO -** Keycloak has full support for Single Sign-On and Single Sign-Out.
- **Admin Console -** Keycloak offers a web-based GUI where you can “click out” all configurations required by your instance to work as you desire.
- **User Identity and Accesses -** Keycloak can be used as a standalone user identity and access manager by allowing us to create users’ databases with custom roles and groups. This information can be further used to authenticate users within our application and secure parts of it based on pre-defined roles.
- **External Identity Source Sync -** In case your client currently has some type of user database, Keycloak allows us to synchronize with such a database. By default, it supports `LDAP` and `Active Directory` but you can create custom extensions for any user database using Keycloak User storage API. Keep in mind that such a solution may not have all data necessary for Keycloak to be fully functional, so remember to check if your desired functionality works.
- **Identity Brokering -** Keycloak can also work as a proxy between your users and some external identity provider or providers. Their list can be edited from Keycloak Admin Panel.
- **Social Identity Providers -** Additionally, Keycloak allows us to use Social Identity Providers. It has built-in support Google, Twitter, Facebook, Stack Overflow but, in the end, you have to configure all of them manually from the admin panel. The full list of supported social identity providers and their configuration manual can be found in Keycloak documentation.
- **Pages Customization -** Keycloak lets you customize all pages displayed by it to your users. Those pages are in `.ftl` format so you can use classic `HTML` markups and `CSS` styles to make the page fit your application style and your company brand. You can even put custom `JS` scripts as part of pages customization so possibilities are limitless.

These are all of Keycloak's features that were present in the documentation. Of course, this tool offers even more possibilities some of which I’ll cover later in the post, which is described in a much more detailed way in the documentation.

So to describe our use case as simply as possible, I'll take a very high-level approach.

![keycloak.png](https://github.com/AnimeshRy/blog/blob/master/assets/images/keycloak/keycloak.png?raw=true)

Assume we have a simple React-based frontend and a Django API backend that needs to communicate with each other.

- The flow would be the frontend communicating with Keycloak which is running on a separate instance for authentication.
- The front end receives an access token from Keycloak.
- Frontend then sends a request to the API backend with the unique access token generated by Keycloak.
- The backend verifies the token and maps our permissions from the token, the requests are then authorized based on the user and returned a valid response.

This would be a common flow for any web application out there. You could implement Single Sign-On but that wasn’t our goal at the moment.

## Was it easy to implement?

Until now everything looked very easy and simple but there were a number of challenges we faced.

1. Custom Login Pages - Keycloak offers its prebuild login components to use for authentication, the HTML-based components are good but they don’t offer a lot of flexibility in terms of customization. You can add CSS-based themes but anything more utilizing javascript was not possible. We wanted our Login Page to work with Keycloak which didn’t have a very straightforward solution.

    ![1_JeXJD_0oySm1r9_obO2YgA.png](https://github.com/AnimeshRy/blog/blob/master/assets/images/keycloak/login.png?raw=true)

    Resolution - Calling the Open ID endpoints from Keycloak for simple login was pretty doable but do implement social authentication with google from our login page, that was a hurdle that overcame by using a slightly less secure Direct Access Grant Flow coupled with utilizing a [kdp_hint](https://wjw465150.gitbooks.io/keycloak-documentation/content/server_admin/topics/identity-broker/suggested.html) parameter in our requests

2. No Python Adapters - As at the time of writing, there aren’t any official Python Keycloak adapters present, there is a react one out there but as we wanted to utilize our custom login pages it was better to call the APIs directly. As for the backend, we ended up writing our own middleware for validating requests and giving authorization permissions.
3. Client Specific Management Consoles - You might not know this but you have to create clients in your realm, these clients are nothing but applications. Be it a frontend client or the backend API, if you want to authenticate from that app, you gotta register it as a client. There is not a single Identity Management Solution out there which provides us with this BUT Keycloak contains a development feature that enabled us to create client-specific admin consoles with fine-grained permission management.

There were more problems we encountered but if carefully skimming through the documentation solved most of them.

## Distributions of Keycloak

Currently, Keycloak has three major distributions.

1. **Server**
The standalone application is downloadable from the Keycloak page in form of a tar or zip archive with all scripts, docs, and assets needed to work normally. As for now, there are two main versions of this distribution: one is powered by WildFly server while the other is powered by Quarks. It is now in the preview stage so some unexpected error may occur.

2. **Docker Image**
Distribution appropriate for Docker, Podman, Kubernetes, and OpenShift. There are two official docker images for Keycloak: one is held in Quay Container Registry - [quay.io/keycloak/keycloak](http://quay.io/keycloak/keycloak), the second one is held in Docker Hub - jboss/keycloak. You can download both of them with a simple docker pull command.

3. **Operator**
Distribution for Kubernetes and OpenShift based on Operator SDK.
As you can see, everybody can find an appropriate distribution. If you use Docker or Kubernetes you have Keycloak image and operator. On the other hand, if you prefer a more conventional deployment type you will also find distribution for you. Even then Keycloak Docker image can be extremely useful for development and testing.

You can set up your test Keycloak server then do changes and test them. After tests you can restart your Docker image and all changes made to your Keycloak will be reverted and you will get a clear environment for further tests. All three distributions can be downloaded from here.

For reference, this was the docker-compose we used

```docker
version: '3.6'

services:
  keycloak:
      image: jboss/keycloak
      environment:
        DB_VENDOR: POSTGRES
        DB_ADDR: "database_address"
        DB_DATABASE: "database_name"
        DB_USER: "db_name"
        DB_PASSWORD: "db_pass"
        KEYCLOAK_USER: "keycloak_username"
        KEYCLOAK_PASSWORD: "keycloak_password"
        # JAVA_OPTS: "-Dkeycloak.profile=preview -Dkeycloak.profile.feature.admin_fine_grained_authz=enabled"
      ports:
        - "8088:8080"
      volumes:
        - ./themes/puretalent:/opt/jboss/keycloak/themes/puretalent
```

## **Why You Should Know Keycloak**

First of all, it is free. You may think that it is funny but in fact, most tools with such features as AuthO or Okta are paid.

Secondly, it supports three different authentication protocols which give you the possibility to cover many applications with different security demands with a single tool.

Additionally, you can choose an authentication protocol based on what you need or what you think will be better for your application and you are not limited by the tool you are using. Keycloak is also an upstream project for Red Hat SSO product so you can be sure that it is a well-written and well-designed system.

Moreover, it has big community support which guarantees that there are a lot of examples of how to do something and that you can count on others to help you with your problems. Keycloak can be very useful when your client has some existing user database like LDAP or Active Directory because it has a built-in mechanism for synchronization with such identity providers.

Additionally, Keycloak supports social identity providers like Google or Facebook straight out of the box so if you want to use Social Login, Keycloak may be very useful for you and your team.

Furthermore, it provides a web-based GUI which makes any configurations changes easier. In the end, thanks to Keycloak SSO support you can facilitate your users’ access to multiple services run by your company.

# **When It May Not Be the Best Choice**

I said why you should use Keycloak so, to stay objective, I will say something about when it may not be the best choice for you.

- Single application with just one client in Keycloak realm – lose all benefits of SSO
- No integration with AD or any other user data provider
- No Social Login
- Pure user database — Keycloak can be used this way but so can a database with specific tables, and it can be much easier to configure if you already have one
- Some kind of enterprise-level guarantees — Keycloak is still an open-source project so you do not have any guarantee provided by its producer about its working or road map and things like customer support are taken care of by Stack Overflow and surely with no hard deadlines for response time

Keep in mind that meeting any or even all of these conditions by your application does not straight away indicate that you should not be using Keycloak. It only means that it may be profitable for you to reconsider the pros and cons of securing your application with Keycloak. Perhaps there is a less complex tool or solution that can be used.

**This article was posted on my office space and is archived here**
