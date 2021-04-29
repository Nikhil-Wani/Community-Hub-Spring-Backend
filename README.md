# Community Hub

### FRONTEND

https://github.com/Nikhil-Wani/Community-Hub-Angular-Frontend

### BACKEND

We will be using different Spring Technologies like Spring Data JPA, Spring Security with JWT Authentication and MySQL as the database. 

### DEPENDENCY 

Lombok, Spring Web, Spring Security, Spring Data JPA, MySQL Java Driver, Java Mail Sender

Let’s now configure the MySQL Database, Hibernate JPA and Java Mail functionality in our application.
Now let’s create domain entities for our Community Hub. In our application, we have Users, who can create Topics and Posts, other users can add Comments on the Posts, and can Vote.
We will set up Spring Security and implement the API to register users in our application. We will also add the functionality to send out emails to the user for Account Activation.

We created the following class under the new package called as config, this class holds the complete security configuration of our application, that’s why the class is named SecurityConfig.java

Following are the contents we configure in the above class.

1.	@EnableWebSecurity:
This is the main annotation which enables the Web Security module in our Project.
2.	WebSecurityConfigurerAdapter:
This is the base class for our SecurityConfig class, it provides us the default security configurations, which we can override in our SecurityConfig and customize them.
3.	Configurations:
Next, we have the configure method which we have overridden from the base class which takes HttpSecurity as an argument.
Here, we are configuring Spring to allow all the requests which match the endpoint “/api/auth/** as these endpoints are used for authentication and registration we don’t expect the user to be authenticated at that point of time.
4.	PasswordEncoder:
Now before storing the user in the database, we ideally want to encode the passwords. One of the best hashing algorithms for passwords is the Bcrypt Algorithm. We are using the BCryptPasswordEncoder class provided by Spring Security.
We have created a RestController and inside this controller, we first created a method that will be invoked whenever a POST request is made to register the user’s in our application. The signup method inside the AuthController is calling another method inside the AuthService class, which is mainly responsible to create the User object and storing it in the database. Now let’s create this AuthService class inside the service package.
Inside the AuthService class, we are mapping the RegisterRequest object to the User object and when setting the password, we are calling the encodePassword() method. This method is using the BCryptPasswordEncoder to encode our password. After that, we save the user into the database. Note that we are setting the enabled flag as false, as we want to disable the user after registration, and we only enable the user after verifying the user’s email address.
Activating new account via email

This brings us to the next section. Now, let us enhance the registration process by only allowing the user to log in after they verify their email. We will generate a verification token, right after we save the user to the database and send that token as part of the verification email. Once the user is verified, then we enable the user to login to our application.

We added the generateVerificationToken() method and calling that method right after we saved the user into UserRepository. Note that, we are creating a random UUID as our token, creating an object for VerificationToken, fill in the data for that object and save it into the VerificationTokenRepository. As we have the token, now it’s time to send an email that contains this verification token.

We need to add some additional dependencies to our project, if we want to send HTML emails from our application, Thymeleaf provides us the template engine, which we can use to create HTML templates and use those templates to send the emails. Let’s add the below thymeleaf dependency to our pom.xml

### An overview of JWT Authentication Mechanism 

1.	So it starts with the Client sending a login request to the server.
2.	The server checks the credentials provided by the user, if the credentials are right, it creates a JSON Web Token (JWT).
3.	It responds with a success message (HTTP Status 200) and the JWT.
4.	The client uses this JWT in all the subsequent requests to the user, it provides this JWT as an Authorization header with Bearer authentication scheme.
5.	When the server, receives a request against a secured endpoint, it checks the JWT and validates whether the token is generated and signed by the server or not.
6.	If the validation is successful, the server responds accordingly to the client.

### Login Flow
 
•	The login request is received by AuthenticationController and is passed on to the AuthService class.
•	This class creates an object of type UserNamePasswordAuthenticationToken which encapsulates the username and password provided by the user as part of the login request.
•	Then this is passed on to AuthenticationManager which takes care of the authentication part when using Spring Security. It implements lot of functionality in the background and provides us nice API we can use.
•	The AuthenticationManager further interacts with an interface called UserDetailsService, this interface as the name suggests deals with user data. There are several implementations that can be used depending on the kind of authentication we want. There is support for in-memory authentication, database-authentication, LDAP based authentication. 
•	As we store our user information inside the Database, we used Database authentication, so the implementation access the database and retrieves the user details and passes UserDetails back to AuthenticationManager.
•	The AuthenticationManger now checks the credentials, and if they match it creates an object of type Authentication and passes it back to the AuthService class.
•	Then we create the JWT and respond back to the user.
Authentication Flow in our application, according to the sequence diagram when the client has authenticated successfully, on each subsequent request, the client provides the JWT to the server and the server should validate it.

### JWT VALIDATION 

•	The client makes a REST call to our API, the token is sent as part of the Authorization header by following the Bearer Scheme.
•	The request is intercepted by the JWTAuthenticationFilter, which is a custom component, this filter class validates JWT, and if the token is valid, the request is forwarded to the corresponding Controller.
•	So the JwtAuthenticationFilter class extends the class OncePerRequestFilter, by extending this class we can implement our validation logic inside the doFilterInternal() method.
•	Inside the doFilterInternal method, we are calling the getJwtFromRequest method which retrieves the Bearer Token (ie. JWT) from the HttpServletRequest object we are passing as input.
•	Once we retrieve the token, we pass it to the validateToken() method of the JwtProvider class.
•	The validateToken method uses the JwtParser class to validate our JWT. If you remember in the previous part, we created our JWT by signing it with the Private Key. Now we can use the corresponding Public Key, to validate the token.
•	Once the JWT is validated, we retrieve the username from the token by calling the getUsernameFromJWT() method.
•	Once we get the username, we retrieve the user using the UserDetailsService class and store the user inside the SecurityContext

Now, we will write APIs to comment on the Posts created by a user. We will also send Comment Notification emails to the creator of the Post.

•	First off, we have our CommentsController class where we have 3 endpoints which we mentioned previously in a table.
•	The createComment takes an object of type CommentsDtos as input, the creation logic is handled inside the CommentService class
•	Once the comment is created we are returning a 201 response back to the client (CREATED).
•	After that we have the getAllCommentsForPost() and getAllCommentsByUser() methods where we are retrieving the relevant comments for given user/post from the CommentService.
•	These methods are returning and OK which means HTTP Status 200.
•	Inside the CommentService class, we have our createComment method where we are creating the comment, and right after that we are sending a notification email to the user through sendCommentNotification()
•	We are using the CommentMapper interface to map our Comment and CommentsDto class.
Notice that when using Instant.now() inside the mapping for the field createDate we are using the fully qualified class name of Instant class. This is because as this expression is inside a String, Mapstruct is not able to recognize and import the relevant import statements for this class

### Update Post API with Vote

When the user queries the Post API to get a single post or all posts. We have to also provide the vote information along with post details.
We added 3 new fields inside the PostResponse.java class – voteCount, commentCount, duration
Now we have to update our PostMapper.java class to map also these new additional fields. One advantage of creating this PostMapper.java is now our mapping logic is decoupled from the actual business logic
•	I changed PostMapper.java from an interface to abstract class, this is because we need information to map the new fields (voteCount, commentCount and duration).
•	As I said before, instead of updating PostService.java class with this logic, we can just change the interface to abstract class and inject the needed dependencies into our class to fill the new field’s information.
•	At line 30 – You can observe that the voteCount is set to a constant value – 0. This is because whenever we want to create a new Post object, we set the default vote count as 0.
•	When mapping from Post to PostResponse, we explicitly need to map only the fields commentCount (handled through commentCount() method) and duration (handled through getDuration())
•	The getDuration() is using a library called TimeAgo. This is a java library that shows us the dates in the relative Time Ago format.
•	When we called the endpoint to create a vote – /api/votes/, we got the response back as 200 OK. Here we have provided the VoteType as DOWNVOTE and id as 3
•	Now we made a call to get All Posts in our application, and for the post with id – 3 we can see that the voteCount is -1.
•	You can also observe that the duration field is shown as 8 days ago instead of the usual timestamp.

### Implementing Logout : 

You are already familiar that JWT (JSON Web Tokens) are mainly used in the authorization flows in the web application. The user first provides the credentials to the server and the server responds back to the client with a JWT if the credentials are correct.
So the client uses this token to authorize itself for all the subsequent requests. Usually, if the client is a web browser, the token is stored in a form of browser storage.
So the obvious thing to do is to delete this token from the browser storage as part of the Logout implementation.

### Is Logout just on the client-side enough?

Imagine if a hacker somehow got access to the token you are using. Then he/she can impersonate the user and make requests to the backend even after the user has chosen to logout.

### How to overcome this :

To overcome this shortcoming, we have below ways to implement Logout/JWT Invalidation in our application.

1)Introduce expiration of JWT
This looks like a valid point, we introduce expiry time for our tokens and after this time, the tokens are no longer valid. We ideally keep this expiration time, Once the token is expired it is no longer valid, forcing the user to authenticate to get a new token. This is terrible user experience and a good way to drive away users from our application.
2) Store JWT inside Database 
The next option is to store the token inside the database. This completely defeats the purpose of using JWT. JWT is by definition stateless and the advantage of using JWT is to bypass the database lookup when authorizing the client.
3) Implement Token Blacklisting
We can use a combination of the above-mentioned approaches to implement Token Blacklisting like below.
When the user log’s out from the browser we delete the token and store this token inside Redis. On each user request, we perform a lookup against Redis and if the token is found inside, we throw an exception.
4) Introduce Refresh Tokens

The next approach is to introduce a concept called Refresh Tokens.When the client first authenticates, the server provides an additional token called a Refresh Token (stored inside our database) additional to the short-lived JWT.
When our JWT is expired or about to be expired, we will use the refresh token to request a new JWT from the server.
In this way, we can keep on rotating the token until the user decides to logout from the application. Once the user logs out, we will also delete the refresh token from the database.
This leaves us with a very short window where the user logs out and the token is still valid.
As you can observe, there is no perfect way to implement logout when using JWT. There are multiple approaches we can choose, but each one of them has there disadvantages.
That is why I strongly recommend using OAuth to implement authentication and authorization in your application. In this way, you need not worry about how to handle user login and logout functionality.
Having said that, let us see how to implement the logout functionality in our application. We will use approach 4 (Refresh Tokens) to implement logout.
Updated Authentication/Authorization Flow
 
We have already explained the existing Authentication/Authorization flow. Now let’s see the new points introduced here.

•	In Step 2, after the authentication is successful, we send a Refresh Token along with the JWT back to the client.
•	In Step 6, the client understands that the JWT is expired or about to be expired and request the server to provide a new JWT by including the Refresh Token inside the request.
•	  In Step 7, the server then verifies the Refresh Token by looking it up in the database and if it matches, generates a new JWT and responds back to the client (Step 8).

Let’s see how to implement the above Refresh Token Concepts in our project.

Step 1: Introducing expiration times for our JWT. As our JWT creation logic is inside JwtProvider.java we will make changes to it.
•	So the first thing you can observe is we have introduced the field jwtExpirationInMillis which is being injected with property value jwt.expiration.time which is set to 900000 ms (15 minutes).
•	We convert this expiration time which is in milliseconds to Date and passing that value to the setExpiration() while generating the JWT.
•	We are also exposing the jwtExpirationInMillis field using the getJwtExpirationInMillis() method.
Step 2:  Implement logic to generate Refresh Tokens. Let’s create a class called RefreshTokenService.java where we can implement the logic to manage our Refresh Tokens.
•	The first method we have is generateRefreshToken() which creates a random 128 bit UUID String and we are using that as our token. We are setting the createdDate as Instant.now(). And then we are saving the token to our MySQL Database.
•	Next, we have validateRefreshToken() which queries the DB with the given token. If the token is not found it throws an Exception with message – “Invalid refresh Token”
•	Lastly, we have deleteRefreshToken() which as name suggests deletes the refresh token from the database.
Step 3: Enhance our login functionality so that it includes the generated Refresh Token inside the AuthenticationResponse.
The update AuthenticationResponse.java looks something like below:

### AuthenticationResponse.java

If you compare the present version with the previous version, you can see that we are building the AuthenticationResponse object with additional fields refresh token and expiration time.
We have also created the refreshToken() method which first validates the token coming from RefreshTokenRequest object. If the validation is successful, we are generating the JWT again and including it in the AuthenticationResponse.

Step 4: Implement endpoints for Refresh Token and Logout.
•	We have to new endpoints /api/auth/refresh/token and /api/auth/logout
•	The Refresh Token API calls the refreshToken method inside the AuthService
•	The Logout API call just calls the deleteRefreshToken method inside the RefreshTokenService

### Testing Time 

First, we will log in to our application and get our JWT, for demonstration purposes I have changed the expiration time of our JWT to 1.5 minutes instead of 15 minutes
You can observe we received authenticationToken, refreshToken and expiresAt and username fields as a response back from the Login API call.
If we use the authenticationToken and make calls to any endpoint, we should receive the response without any errors. Once the expiration time is reached, if we try to call the /api/posts/ endpoint, we will receive an error message as shown in below
Now let’s make a call to get new JWT using the refresh token. You can see that we received a new JWT along with the refresh token we included in the request. Now let’s try to call the Logout endpoint and then try to Refresh the JWT. So you can observe when we tried to refresh our JWT, we received an error  “Invalid refresh Token” because our Refresh Token was already deleted.

