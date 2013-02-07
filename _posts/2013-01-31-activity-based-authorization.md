---
layout: post
title: Activity Based Authorization
---

Activity Based Authorization
============================

Activity based authorization is a great design pattern. It can save you a lot of work, and significantly increase the flexibility of your authorization system. Imagine data driven authorization: fully configurable without changing any source code. To get a better understanding of the concept, [read this first](http://lostechies.com/derickbailey/2011/05/24/dont-do-role-based-authorization-checks-do-activity-based-checks/ "Don’t Do Role-Based Authorization Checks; Do Activity-Based Checks").


Three things are required to successfully implement activity based authorization:
---------------------------------------------------------------------------------

1. A mapping of roles to activities (usually in a database)
2. A way of specifying which activities require authorization
3. A mechanism to authorize a user for a given activity assuming the above


A mapping of roles to activities
--------------------------------

A role represents a collection of activities. It saves you having to tie activities directly to a user. By decoupling activities and users via roles, you are able to add activities to roles on the fly.

The application itself never deals with roles. The application only specifies activities. It is up to the activity system to check whether the activity is associated with one of the user's roles.


A way of specifying which activities require authorization
----------------------------------------------------------

In C# this could be implemented via attributes. The most flexible solution in this case is a custom attribute `[AuthActivity]` which has the following characteristics:

* The `[AuthActivity]` attribute can only be applied to methods (as opposed to classes)
* The `[AuthActivity]` attribute requires a string parameter specifying the Activity
  * e.g. To enable authorization on the `NewUser` and `UpdateUser` methods:

``` csharp
[AuthActivity(Activity = "CreateUser")]
public void NewUser( ... )
[AuthActivity(Activity = "UpdateUser")]
public void UpdateUser( ... )
```

Note here that the `NewUser()` method maps to an activity called `CreateUser`, while the `UpdateUser()` method maps to an activity called `UpdateUser`. You're probably saying to yourself _"Couldn't we just let the attribute to get the name of the method at runtime to save us some typing if the method has the same name as the activity?"_. [Unfortunately not](http://stackoverflow.com/questions/2168942/how-do-i-get-the-member-to-which-my-custom-attribute-was-applied/2169373#2169373).

While having to explicitly specify the activity name for each method may seem like a violation of the [DRY principle](http://en.wikipedia.org/wiki/Don't_repeat_yourself "Don't repeat yourself"), the added clarity and readability outweighs the few extra characters you have to type. It also saves you from inadvertently breaking authorization if you later decide to change the method name.


A mechanism to authorize a user for a given activity
----------------------------------------------------

As above, the attribute implementation would take the Activity name, the user and the inferred role based on the user and check to see if that role mapped to the specified Activity:

``` csharp
public void AuthActivityAttribute(string Activity)
{
    User currentUser = GetCurrentUser();

    if( GetUserActivities( currentUser ).Contains( Activity ) )
    {
        // Authorized
    }
    else
    {
        // Unauthorized
    }
}

public List<Activity> GetUserActivities(User currentUser)
{
    List<Role> roles = GetUserRoles( currentUser );
    List<Activity> activities = new List<Activity>();

    foreach(Role role in roles)
    {
        List<Activity> roleActivities = GetRoleActivities( role );
        activities.AddRange( roleActivities );
    }

    return activities;

    // If we wanted to be concise, this whole method could be written as:
    // return GetUserRoles( currentUser ).SelectMany( x => x.GetRoleActivities( x ) );
}
```


What is an activity?
--------------------

The mental model I’m currently using maps CRUD operations on entities to activities. For example:

* CreateUser
* ReadUser
* UpdateUser
* DeleteUser

This assumes a relational database is being used. Using the above mental model makes it easy to define activities without any ambiguities.