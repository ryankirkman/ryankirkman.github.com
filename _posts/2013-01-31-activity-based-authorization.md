---
layout: post
title: Activity Based Authorization
---

# {{page.title}}

Activity based authorization is an amazing concept. It can save you a lot of work, and significantly increase the flexibility of your authorization system. Imagine data driven authorization: fully configurable authentication without changing any source code. To get a better understanding of the concept, [read this first](http://lostechies.com/derickbailey/2011/05/24/dont-do-role-based-authorization-checks-do-activity-based-checks/ "Don’t Do Role-Based Authorization Checks; Do Activity-Based Checks").


### Three things are required to successfully implement activity based authorization:

  1. A mapping of roles to activities (usually in a database)
  2. A way of specifying which activities require authorization
  3. A mechanism to authorize a user for a given activity assuming the above

## A mapping of roles to activities

A role represents a collection of activities. It saves you having to associate activities directly to a user. By decoupling activities and users via roles, you are able to add activities to roles on the fly.


## A way of specifying which activities require authorization

In C# this could be implemented via attributes. The most flexible solution in this case is a custom attribute [Authorize] which has the following characteristics:

* When [Authorize] is a applied to a method, the attribute gets the method name via reflection and uses this as a lookup for the activity name
	* e.g. To authorize for the UpdateUser Activity:

	``` csharp
	[Authorize]
	public void UpdateUser( … )
	```

* When the activity name is different to the method name, the [Authorize] attribute may take an additional string parameter specifying the Activity
	* e.g. To authorize a method for the CreateUser Activity:

	``` csharp
	[Authorize(Activity = “CreateUser”)]
	public void CreateNewUser( … )
	```

* When the [Authorize] attribute is applied to a class, it acts as if each public method of the class has the [Authorize] attribute specified
	* This requires that all method names map directly to Activity names


## A mechanism to authorize a user for a given activity

As above, the attribute implementation would take the Activity name, the user and the inferred role based on the user and check to see if that role mapped to the specified Activity:

``` csharp
public void AuthorizeAttribute(string Activity) {
    User currentUser = GetCurrentUser();
    Role userRole = GetRoleForUser(currentUser);

    if(GetActivitiesForRole(userRole).Contains(Activity) == false)
    {
        // The user is unauthorized for this activity
    }
}
```


## What is an activity?

The mental model I’m currently using maps CRUD operations on entities to activities. For example:

* CreateUser
* ReadUser
* UpdateUser
* DeleteUser

This assumes a relational database is being used. Using the above mental model makes it easy to define activities without any ambiguities.