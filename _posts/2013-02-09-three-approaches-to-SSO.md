---
layout: post
title: Three Approaches to SSO
---

Three Approaches to SSO
=======================

In this article, we'll explore 3 different approaches to Single Sign-On (SSO). It is assumed that the reader is working at a company that has many applications, and would like a unified way for their users to authenticate into their many applications. Ideally, the same credentials would be used to authenticate across all applications.

They are:

1. __Don't do SSO__ - each application manages authentication and authorization independently
2. __Thin SSO__ - SSO manages authentication and applications independently manage authorization
3. __Heavy SSO__ - SSO manages authentication and authorization


Don't do SSO
------------

This is the easiest option, but is a classic violation of the [DRY principle](http://en.wikipedia.org/wiki/Don't_repeat_yourself "Don't repeat yourself"). Each application manages authentication and authorization separately.

__Advantages__

* Quick and easy

__Disadvantages__

* Violates the [DRY principle](http://en.wikipedia.org/wiki/Don't_repeat_yourself "Don't repeat yourself") - you'll end up duplicating authentication and authorization for every new application you develop
* Requires users to have a separate set of credentials for each application
* Makes integration of applications extremely difficult
  - Because you have no way of matching up users between applications, developing things like a dashboard with pre-authenticated links to all a user's available applications would be extremely difficult


Thin SSO
--------

Thin SSO appears to be the way most SSO systems are designed. For example, your Google or Facebook account. You log in once and are authenticated on all sites that support either Google or Facebook SSO, but each application independently manages authorization.

This is probably the most practical approach to SSO. The only assumption it makes is that a user is the same across all applications. It leaves authorization up to each application as we are assuming an authorization role requires the context of an application to give it meaning.

__Advantages__

* Practical - doesn't make any assumptions about roles or authorization
* Developing a dashboard is simplified
  * e.g. you could query each application to see if a user is present and then aggregate the results to present an iPhone-esque home screen with all a user's applications
* Flexible - each application has full control over authorization

__Disadvantages__

* Potentially duplicated work if many applications share the same authorization roles and structure


Thick SSO
---------

This is the most extreme option, where both authentication _and_ authorization are handled by the SSO system. The application can query the SSO system for a user's roles, but it has no control over them. The job of the application is to interpret the roles within the context of the application.

__Advantages__

* Can save a significant amount of work if all roles are applicable to all applications

__Disadvantages__

* Poor flexibility - all applications must share the same concept of roles
  * High cost of change if you need to introduce an application that differs in its concept of roles