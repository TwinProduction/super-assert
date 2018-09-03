# super-assert

Heavily inspired by Spring Framework's 
[Assert](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/util/Assert.html) 
class in `org.springframework.util` and JUnit's assertion library, SuperAssert's goal is to provide a complete
 one-line assertion solution with custom exception support.

By providing a one-line solution, developers significantly enhance the readability of their code by reducing 
the amount of lines taken by minor, but sometimes numerous checks like this:

```java
if (username == null || username.length == 0) {
	throw new Exception("...");
}
```

and instead, use an elegant one-line solution like this:

```java
SuperAssert.isTrue(username == null || username.length == 0, "...");
```

In fact, SuperAssert shines even more when you want to give more details to each specific error cases:

```java
SuperAssert.notNull(username, "Your username cannot be null");
SuperAssert.isFalse(username.isEmpty(), "Your username cannot be blank");
SuperAssert.isAlphanumeric(username, "Your username must contain only alphanumeric characters");
SuperAssert.isNull(getUserByUsername(username), "The username you selected is already taken by another user");
SuperAssert.isAscii(password, "Your password cannot use illegal characters");
```

As this project is currently in progress, you're welcome to contribute.


## Usage

By default, SuperAssert will throw an `IllegalArgumentException` with the message passed as parameter.
It's very important to understand that `IllegalArgumentException` is an **unchecked exception**, meaning that 
using SuperAssert does not require you to declare an exception in the method's or in the constructor's `throws` clause
**unless you specify a custom checked exception**.


### Without custom exception

```java
SuperAssert.notNull(object, "The object passed as parameter cannot be null");
```


### With custom exception

```java
SuperAssert.notNull(object, new CustomException("The object passed as parameter cannot be null"));
```

----------------------

## Real world examples

```java
public User getUserById(Long id) {
	SuperAssert.notNull(id, "A user cannot have a null id.");
	return userRepository.findById(id).orElse(null);
}


public User createUser(String username, String password) {
	SuperAssert.notNull(username, "Username cannot be null");
	SuperAssert.notNull(password, "Password cannot be null");
	SuperAssert.isAlphanumeric(username, "Username can only contain the following: a-z A-Z 0-9");
	SuperAssert.isAscii(password, "Password contains invalid character");
	SuperAssert.isNull(getUserByUsername(username), "Username is already taken");
	return userRepository.save(new User(username, hash(password)));
}
```

----------------------


## Current API


### notNull

**Syntax**: `SuperAssert.notNull(object, message | exception)`

Asserts whether an object is not null. 
- If it is null, an exception will be thrown
- If it isn't null, `true` will be returned 

#### Example

This will throw an exception, because the assertion that the `user` is `notNull` is **wrong** _(the user is null)_:
```java
User user = null;
SuperAssert.notNull(user, "User cannot be null"); 
```
This will return `true`, because the assertion that the `user` is `notNull` is **correct** _(the user is not null)_:
```java
User user = new User();
SuperAssert.notNull(user, "User cannot be null"); 
```


### isNull

**Syntax**: `SuperAssert.isNull(object, message | exception)`

Asserts whether an object is null. 
- If it is not null, an exception will be thrown
- If it is null, `true` will be returned 

#### Example

This will throw an exception, because the assertion that the `user` `isNull` is **wrong** _(the user is not null)_:
```java
User user = new User();
SuperAssert.notNull(user, "User already exists"); 
```
This will return `true`, because the assertion that the `user` `isNull` is **correct** _(the user is null)_:
```java
User user = null;
SuperAssert.notNull(user, "User already exists"); 
```


### isTrue

**Syntax**: `SuperAssert.isTrue(object, message | exception)`

Asserts whether a boolean is true. 
- If it is `true`, `true` will be returned
- If it is `false`, an exception will be thrown

#### Example

This will throw an exception, because the assertion that the `title.isEmpty()` condition `isTrue` is **wrong** _(the title is not empty)_:
```java
String title = "Some pretty cool title";
SuperAssert.isTrue(title.isEmpty(), "Title cannot be empty"); 
```
This will return `true`, because the assertion that the `title.isEmpty()` condition `isTrue` is **correct** _(the title is empty)_:
```java
String title = "";
SuperAssert.isTrue(title.isEmpty(), "Title cannot be empty"); 
```


### isFalse

**Syntax**: `SuperAssert.isFalse(object, message | exception)`

Asserts whether a boolean is false. 
- If it is `true`, an exception will be thrown
- If it is `false`, `true` will be returned

#### Example

This will throw an exception, because the assertion that the `locked` condition `isFalse` is **wrong**:
```java
boolean locked = true;
SuperAssert.isFalse(closed, "The door is locked, you can't get in!"); 
```
This will return `true`, because the assertion that the `locked` condition `isFalse` is **correct**:
```java
boolean locked = false;
SuperAssert.isFalse(closed, "The door is locked, you can't get in!"); 
```


### notEmpty

**Syntax**: `SuperAssert.notEmpty(object, message | exception)`

Asserts whether a list is not empty. 
- If it is empty, an exception will be thrown
- If it is not empty, `true` will be returned

#### Example

This will throw an exception, because the assertion that the list of `users` is `notEmpty` is **wrong**:
```java
List<User> users = new ArrayList<>();
SuperAssert.notEmpty(users, "The user list cannot be empty!"); 
```
This will return `true`, because the assertion that the list of `users` is `notEmpty` is **correct**:
```java
List<User> users = Arrays.asList(user1, user2, ...);
SuperAssert.notEmpty(users, "The user list cannot be empty!"); 
```


### notEmptyOrNull

**Syntax**: `SuperAssert.notEmptyOrNull(object, message | exception)`

Asserts whether a list is not empty or null. 
- If it is empty or null, an exception will be thrown
- If it is not empty or null, `true` will be returned

_In essence, this method is just an aggregation of `notNull` and `notEmpty`._

#### Example

This will throw an exception, because the assertion that the list of `users` is `notEmptyOrNull` is **wrong** _(the user list is null)_:
```java
List<User> users = null;
SuperAssert.notEmpty(users, "The user list cannot be empty or null!"); 
```
This will return `true`, because the assertion that the list of `users` is `notEmptyOrNull` is **correct**:
```java
List<User> users = Arrays.asList(user1, user2, ...);
SuperAssert.notEmpty(users, "The user list cannot be empty or null!"); 
```


### isAscii

**Syntax**: `SuperAssert.isAscii(string | char, message | exception)`

Asserts whether a character/string is a _printable_ ASCII. 
- If it is not a printable ASCII character, an exception will be thrown
- If it is, `true` will be returned

**NOTE**: _While the name of this method is `isAscii`, be aware that this assumes **PRINTABLE** 
ASCII characters. In other words, characters whose value is between 32 and 126 inclusively._

#### Example

This will throw an exception, because the assertion that the string `message` `isAscii` is **wrong**:
```java
String message = "Well,\n this is pretty cool!";
SuperAssert.isAscii(message, "Invalid characters"); 
```
This will return `true`, because the assertion that the string `message` `isAscii` is **correct**:
```java
String message = "Well, This is pretty cool!";
SuperAssert.isAscii(message, "Invalid characters");
```


### isAlphanumeric

**Syntax**: `SuperAssert.isAlphanumeric(string | char, message | exception)`

Asserts whether a character/string is alphanumeric. 
- If it is not alphanumeric, an exception will be thrown
- If it is, `true` will be returned

#### Example

This will throw an exception, because the assertion that the string `username` `isAlphanumeric` is **wrong**:
```java
String username = "John Doe";
SuperAssert.isAlphanumeric(username, "Username must be alphanumeric (a-z A-Z 0-9)"); 
```
This will return `true`, because the assertion that the string `username` `isAlphanumeric` is **correct**:
```java
String username = "J0hnD03";
SuperAssert.isAscii(username, "Username must be alphanumeric (a-z A-Z 0-9)");
```


----------------------

## Using custom exceptions

Using custom exceptions involves passing the exception message as parameter to your custom exception and
then passing that custom exception as parameter instead of the message in the SuperAssert method.

e.g.
```java
SuperAssert.notNull(user, "User cannot be null"); 
```
translates to 
```java
SuperAssert.notNull(user, new CustomException("User cannot be null")); 
```

----------------------

## TODO

**NOTE**: The naming is not fixed. If you have a better idea, feel free to use it.

- hasMoreThanMinLength
	- `hasMoreThanMinLength(str, minLength, message)`
	- `hasMoreThanMinLength(str, minLength, customException)`
- hasLessThanMaxLength
	- `hasLessThanMaxLength(str, maxLength, message)`
    - `hasLessThanMaxLength(str, maxLength, customException)`
- hasBetweenMinAndMaxLength
	- `hasBetweenMinAndMaxLength(str, minLength, maxLength, message)`
	- `hasBetweenMinAndMaxLength(str, minLength, maxLength, customException)`
	
