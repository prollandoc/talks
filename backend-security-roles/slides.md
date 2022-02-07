---
title: Backend security roles
---

# Backend security roles

Pierre Rolland

30 Mar. 2022

![](resources/security-guy.png)

---

## What is it about?

- Quickly explain how access control work, globally, in Symfony
- What we're doing now on the platform
- What are the traps to avoid
- What are the old habits to break

---

## Access control in Symfony

----

### What are the roles?

A role is nothing more than a string beginning with "ROLE_", that we can assign to a user. 
They're used to restrain access to some parts of the application only to the users having the accurate role (ROLE_ADMIN, ROLE_WRITER, etc.)

----

### How do we use them to restrain access?

Several ways:
- In the "access_control" section
- With the "is granted" helpers

----

#### access_control section

Used to restrain access to some routes to users having a given role, defined in `security.yml`

```yaml
# app/config/security.yml (at OC)

security:
  access_control:
  	- { path: ^/admin/, roles: [ROLE_ADMIN] }
```

----

#### "Is granted" helpers

Used to restrain access to a programmatical piece of logic, beyond access control

```php
if ($this->security->isGranted('ROLE_ADMIN')) {
```


Or in a controller

```php
/**
 * @Security("is_granted('ROLE_ADMIN')")
 **/
class MySecureController
```

or

```php
$this->denyAccessUnlessGranted('ROLE_ADMIN');
```

----

#### Special OC bonus
OC has a special annotation that has been created to secure use cases specifically, and relies on the same mechanism:

```php
use OpenClassrooms\UseCase\Application\Annotations\Security;

public class ChangePassword implements UseCase
{
	/**
	 * @Security(roles={"ROLE_ADMIN"})
	 */
	public function execute(ChangePasswordRequest $request): void
	{
		// ...
	}
}
```

----

### Roles can be organized in a hierarchy

```yaml
security:
  role_hierarchy:
	ROLE_ADMIN: ROLE_USER
	ROLE_SUPER_ADMIN: ROLE_ADMIN
```

----

### What if roles are not enough to decide if a user can access a resource?

----

### A voter can express its opinion

```php
class MyVoter extends Voter
{
    protected function supports(string $attribute, $subject): bool
    {
    }

    protected function voteOnAttribute(string $attribute, $subject, TokenInterface $token): bool
    {
    }
}
```

----

### Voters are registered in a catalog

When deciding if a resource can be accessed, the manager traverses all the catalog and first calls `supports` to know if the voter is concerned, then `voteAnAttribute` to know its decision

----

### Strategies

- Affirmative: ONE voter says yes and access is granted
- Consensus: A MAJORITY of concerned voters say yes and access is granted
- Unanimous: ALL concerned voters say yes and access is granted

---

## At OpenClassrooms now

- Scopes
- Actors
- Use case roles
- Use case validation
- Bastard roles

----

### A scope is a category of API endpoints

- API clients (partners, iOS, web, etc.) credentials come with a set of allowed scopes, giving access to some routes
- Used on controllers / OpenAPI

----

### An actor is a Symfony security role that we give to real users reflecting their position on the website

- Has to be used nowhere in the code than in security.yml or voters
- Used with roles hierarchy mechanism

----

### A use case role is a Symfony security role that we place only on use cases

- It is named after the use case (ROLE_DELETE_USER)
- Only the users having an **actor** role **inheriting** this role can access the use case
- A user with a given actor role can execute all the use cases assigned to this actor role

----

#### Example

```yaml
# app/config/security.yml

security:
  role_hierarchy:
	ROLE_ACTOR_ADMIN: ROLE_UPDATE_USER_PASSWORD ROLE_UPDATE_STUDENT_INFORMATION
```

```php
# src/OC/BusinessRules/UseCases/User/UpdateUserPassword.php

class UpdateUserPassword implements UseCase
{
	/**
	 * @Security(roles={"ROLE_UPDATE_USER_PASSWORD"})
	 */
	public function execute(UpdateUserPasswordRequest $request): void
}
```

----

### A use case validation is a Symfony security role that we place only on use cases

- It requires more logic to work than just assigning a role to some actors
- It is named after the use case (CAN_DELETE_USER)
- It comes with a voter to determine if the user can or can not access the use case

----

#### Example

```php
class UpdateUserPassword implements UseCase
{
	/**
	 * @Security(roles={"CAN_UPDATE_USER_PASSWORD"}, checkRequest=true)
	 * OR
	 * @Security(roles={"CAN_UPDATE_USER_PASSWORD"}, checkField=userId)
	 */
	public function execute(UpdateUserPasswordRequest $request): void
}
```

----

### A bastard role is a role that is convenient but...

- ... that does not really respect these new guidelines.
- Example: `CURRENT_USER` which is a use case validation coming with its voter in order to check that the user id corresponds to the logged in user id

---

## Some traps to avoid

----

### Misunderstood strategy

Our strategy is **AFFIRMATIVE**. 

> If you provide several roles, respecting only one of them is enough to access the resource.

----

#### ❌ This grants access to the current user OR any other user having the use case role (potential security breach if too many users have the role)

```php
/**
 * @Security(roles={"ROLE_SUBSCRIBE_NEWSLETTER", "CURRENT_USER"}, checkField="userId")
 */
``` 

----

#### ❌ This does not respect the guidelines (no usecase-dedicated role)

```php
/**
 * @Security(roles={"CURRENT_USER"}, checkField="userId")
 */
```

----

#### ✅ This grants access to the current user having the use case role and respects the guidelines

```php
/**
 * @Security(roles={"CAN_SUBSCRIBE_NEWSLETTER"}, checkField="userId")
 */
```

```php

class CanSubscribeNewsletterVoter extends AbstractCurrentUserVoter
```

----

### Mistaken hierarchy direction


❌ This does not grant every user the right to subscribe to newsletters

```yaml
ROLE_SUBSCRIBE_NEWSLETTER: ROLE_USER
```

✅ This would

```yaml
ROLE_USER: ROLE_SUBSCRIBE_NEWSLETTER
```

----

### Changing roles may imply logging the user out

> That's why refactoring the security roles must be done in a two-steps process, it's ok to keep legacy for a while

---

## Old habits

----

### Registering all the use case roles in `security.yml`

```yaml
ROLE_SUBSCRIBE_NEWSLETTER: ROLE_USER
```

- Confusing
    -  _We can understand "all the users have this right"_
- Unnecessary. 
	- _We can assign "unregistered" roles to actors_

----

### Registering all the use case roles in PHP classes

```php
public const ROLE_SUBSCRIBE_NEWSLETTER = 'ROLE_SUBSCRIBE_NEWSLETTER';
```

- Remainder from the past / mimetism
    -  _Formerly used when actors roles were on use cases_
- Unnecessary. 
	- _You'll see this piece of code remains unused_
- Ok for actors roles / use case validations (CAN_)
	- _Which may be used in voters_

----

### Using `has_role`

`has_role` has been replaced by `is_granted`, and marked as deprecated, in Symfony 4.2

---

## Is it the end of `CURRENT_USER`?

----

### As a scope

- It's not a proper scope
- It's strongly tied to the shape of the path (that must contain `{userId}`)

----

### As a use case role

- It's not a use case role either (not assignable to an actor)

----

### As a use case validation

- It's a disguised use case validation
- We already have a system for those: voters
- There's a `AbstractCurrentUserVoter` that can be extended
	- The `voteOnAttribute` is implemented

----

### Benefits / Drawbacks

- ✅ Don't repeat voters with same purpose
- ✅ Don't overload the decision manager
- ❌ Relies on internal user ID and not public one
- ❌ Not respecting conventions
- ❌ Not automatable
	- _End goal being to get rid of the security annotation_

---

## Where are we currently regarding use cases security?

----

### Amount of use cases respecting the security guidelines

**55** use cases out of 531

_have a ROLE_USE_CASE or a CAN_USE_CASE security_ 

----

### Use cases responding with personal information

**3** use cases out of 70 responding with personal data have an accurate security role (_ROLE_USE_CASE or CAN_USE_CASE or CURRENT_USER_)

> ⚠️ 167 use cases have unguessable responses from type-hinting or PHPdoc, so these numbers may be bigger


---

## What's left to do?

Following actions require fixing the current code base

----

### Step 1: make tests (or lints) blocking if security isn't on use cases

----

### Step 2: Automate use cases security

> Less configuration, more focus

---

## Questions ? :)
