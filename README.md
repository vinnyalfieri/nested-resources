#Rails - Nested Resources

###After this lecture you will learn
* what a nested resource is
* when to use nested resources
* how to create a nested resource
* how to create CRUD actions for the nested resource
  

####You are used to seeing routes like:

```
/users
/users/new
/profile
/profile/new

/users/1
/users/1/edit
/profile/1
/profile/1/edit
```
If a profile exists, shouldn't it belong to a user? Practially all websites you have been to do not use the above route for a profile. They use something like

```
/users/1/profile
#or
/espn.go.com/nfl/schedule
```
Nested resrouces display the relationship in our routes.
