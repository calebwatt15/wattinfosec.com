Title: Authorization Checks Should Use Different Passwords
Date: 12-13-2015
Category: WebApp Security
Tags: WebApp Security
Slug: authorization-checks-should-use-different-passwords
Author: Caleb
Summary: When testing authorization and account changes between two accounts, you should use two different passwords, in order to be able to tell when an account has been taken over, versus when some other change has talen place. A different password can help determine what has changed during an assessment.

Insufficient Authorization
--------------------------
In web apps, every once in a while a user can change another user's account details by intercepting a request to change their own details, and modify some parameter in the request. This is often accomplished by modifying either an ID number, a username, or some other ID. In many cases, this means that an attacker may be able to create an account, then alter another account's password by trying to change their own, capture the request, and modify a username that is otherwise marked uneditable on the front end. 

In Real Life
------------
In a recent assessment, I ran accross this sort of problem. My two users were setup as follows:

![Breakdown of user information and permissions](/images/authCheckPasswordsDifferent01.png)
The web application in question used an email address as the login username. This could not be changed once it was set. Even administrators were forbidden from editing any users' email addresses. In this case, the Account Details page has the "Email" field marked "Disabled", so that it could not be edited. If a user captured the request in a proxy, they could then edit the Email.

![A user can email any field when capturing the request](/images/authCheckPasswordsDifferent03.png)

This lead to an interesting find. As a standard user, named "standard@test.com" I set my email to the admin user, named "admin@test.com". This let me change my email to the admin's login. 

![StandardUser changes their email to admin@test.com](/images/authCheckPasswordsDifferent04.png)

Sadly, when I tried to login with "admin@test.com" and standard's password, it failed. However, if I tried to authenticate using "admin@test.com" and the normal admin password, it worked. 

Given this I assumed that the edit did not work since, admin@test.com was already a user, however I was wrong. If the admin email was input, with the admin password, then the application would authenticate the user, but it had the name and permissions of the standard user. This, in effect, forced the admin to have lower permissions, denying that admin the ability to perform admin duties. The admin had been forced into the standard user permissions, with the first and last name of the standard user.

![The Standard user's email and password were overwritten by the admin email and password, without any knowledge of the admin password](/images/authCheckPasswordsDifferent02.png)
I may not have realized this as quickly, had I used the same password for both users. I would have assumed that the lower user took over the Admin account, as it would have been the lower user's password, name, and permissions. As is, this specific application allowed me to give another user my account, but it still had the proper username and password that they originally set. This would have been much harder, if even possible to notice without a different password for each user.

TL;DR
----
Set all test accounts to different passwords, as during tests you will be more easily able to determine if a forced account change changed a user, or took over another account.
