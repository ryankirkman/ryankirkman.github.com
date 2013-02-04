---
layout: post
title: Activity Based Authorization
---

Activity Based Authorization
============================

Activity based authorization is an amazing concept. It can save you a lot of work, and significantly increase the flexibility of your authorization system. Imagine data driven authorization: fully configurable without changing any source code. To get a better understanding of the concept, [read this first](http://lostechies.com/derickbailey/2011/05/24/dont-do-role-based-authorization-checks-do-activity-based-checks/ "Don’t Do Role-Based Authorization Checks; Do Activity-Based Checks").


Three things are required to successfully implement activity based authorization:
---------------------------------------------------------------------------------

1. A mapping of roles to activities (usually in a database)
2. A way of specifying which activities require authorization
3. A mechanism to authorize a user for a given activity assuming the above


A mapping of roles to activities
--------------------------------

A role represents a collection of activities. It saves you having to associate activities directly to a user. By decoupling activities and users via roles, you are able to add activities to roles on the fly.


A way of specifying which activities require authorization
----------------------------------------------------------

In C# this could be implemented via attributes. The most flexible solution in this case is a custom attribute `[AuthActivity]` which has the following characteristics:

* When `[AuthActivity]` is a applied to a method, the attribute gets the method name via reflection and uses this as a lookup for the activity name
  * e.g. To AuthActivity for the UpdateUser Activity:

``` csharp
[AuthActivity]
public void UpdateUser( ... )
```

* When the activity name is different to the method name, the `[AuthActivity]` attribute may take an additional string parameter specifying the Activity
  * e.g. To AuthActivity a method for the CreateUser Activity:

``` csharp
[AuthActivity(Activity = “CreateUser”)]
public void NewUser( ... )
```

* When the `[AuthActivity]` attribute is applied to a class, it acts as if each public method of the class has the `[AuthActivity]` attribute specified
  * This requires all method names map directly to Activity names

``` csharp
[AuthActivity]
public class UserController
{
	public void CreateUser( ... ) { ... }
	public void UpdateUser( ... ) { ... }
	public void DeleteUser( ... ) { ... }
}
```

* If both the class and method have the `[AuthActivity]` attribute, the method attribute has a higher precedence than the class attribute
  * This saves you having to decorate each method in a class if all methods except 1 directly map to activities:

``` csharp
[AuthActivity]
public class UserController
{
	public void CreateUser( ... ) { ... }
	public void UpdateUser( ... ) { ... }
	[AuthActivity(Activity = "DeleteUser")]
	public void RemoveUser( ... ) { ... }
}
```

A mechanism to authorize a user for a given activity
----------------------------------------------------

As above, the attribute implementation would take the Activity name, the user and the inferred role based on the user and check to see if that role mapped to the specified Activity:

``` csharp
public void AuthActivityAttribute(string Activity) {
    User currentUser = GetCurrentUser();
    Role userRole = GetRoleForUser(currentUser);

    if(GetActivitiesForRole(userRole).Contains(Activity))
    {
        // Authorized
    }
    else
    {
        // Unauthorized
    }
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