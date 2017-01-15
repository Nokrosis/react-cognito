# react-cognito

## Installation

Install the prerequisites:

    sudo apt install npm

Check out the code somewhere:

    git clone git@github.com:isotoma/react-cognito.git

Change into that directory, and then run the npm installer:

    npm install

## Running the example

The package `local-web-server` is included as a development dependency, so if
you have installed the development packages you should have the `ws` command in
`node_modules/.bin`.

First build the library, and then the examples, then change into the htdocs directory and run the webserver:

    webpack -d
    webpack -d --config webpack.examples.js
    cd examples/htdocs
    ws -s index.html

The `-s` means all requests are sent to the single page application.

## Trying it out

### Set up your user and identity pools

TBC

### Try it out

#### Create a test user

1. Go to your user pool and go to "Users and groups"
2. Click "create user"
3. Complete the form:
  - Enter a username 
  - Enter a password that conforms to the rules of the user pool
  - Uncheck 'Mark phone number as verified?'
  - Uncheck 'Mark email as verified?'
  - Enter a valid email address
  - Click 'Create User'

#### First time login

1. Go to your deployed example application webserver
2. You should see the login form
3. Enter the username and password created above
4. You should be asked for a new password, since this is your first login
5. Enter a new password. It must conform to the rules of the user pool.
6. You should then be taken to a verification code entry screen.  check your email and enter the code.
7. You should now see the logged in screen, showing your attributes and giving you some options.

Note that during this flow quitting and reloading the browser, then navigating back to this page, 
will result in you returning to the correct step.

#### Logout and Login

1. Click 'log out'
2. You should see the login form
3. Login again using your username and the new password you chose above.
4. You should be taken to the logged in screen

#### Password changing

1. Click 'Change password'
2. Enter your existing password and a new password
3. Click set new password
4. you should see a message saying your password has been changed
5. Click 'Home'

#### Change email address

1. Click 'Change email address'
2. You should see a form with your existing email address
3. Enter a new valid email address
4. You should be asked for a verification code.  Check your email for the code.
5. Enter the code.
6. You will be taken to the logged in screen.

Note that you can also reload the page after step 4, or close your browser, and you will 
be required to enter the verification code.

#### Password recovery

1. Log out
2. Click 'Password Reset'
3. Enter your username
4. Click 'send verification code'
5. It should say 'verification code sent'
6. Check your email and get the code
7. enter the code and your new password
8. Click 'set new password'
9. It should say 'Password reset'
10. Click 'Home'
11. Log in with your new password


## Summary of use cases

The cognito js library helpfully lists all of the 24 use cases [on their github](https://github.com/aws/amazon-cognito-identity-js/).

This is the current status of each use case in react-cognito:

### Completed

- UC1 Registering a user with the application
- UC2 Confirming a registered, unauthenticated user
- UC3 Resending a confirmation code via SMS
- UC4 Authenticating and establishing a session
- UC5 Retrieving user attributes for an authenticated user
- UC6 Verify email address for an authenticated user
- UC8 Update a user attribute for an authenticated user
- UC11 Change the current password for an authenticated user
- UC12 Starting and completing a forgotten password flow for an unauthenticated user
- UC14 Sign out
- UC16 Retrieve the user from local storage
- UC17 Log into an identity pool with a cognito user
- UC23 Set a new password on inital login for an admin created user

### Planned for version 1, but not yet implemented

### Not planned for version 1

#### Trivial anyway

- UC7 Delete a user attribute for an authenticated user
- UC9 Enable MFA for a user on a pool that has optional MFA
- UC10 Disable MFA for a user on a pool that has optional MFA
- UC13 Deleting an authenticated user
- UC15 Global sign out (invalidates all issued tokens)

#### MFA Support

- UC24 Retrieve the MFA options for the user in case MFA is optional

Plus any MFA implementation

#### Device support

- UC18 List all remembered devices for an authenticated user
- UC19 List all information about the current device
- UC20 Remember a device
- UC21 Do not remember a device
- UC22 Forget the current device

## Issues

- Review how visual transitions should be integrated into e.g. logging in
- Consider offline / liefi use

# The Redux Model

## State Machine

Users start in state `LOGGED_OUT`.

When they enter a username and password we call `authenticateUser` on the CognitoUser
object and transition to a new state if the credentials worked:

- `MFA_REQUIRED`
- `NEW_PASSWORD_REQUIRED`
- `CONFIRMATION_REQUIRED`
- `AUTHENTICATED`

If in `AUTHENTICATED` then a transition happens using a store subscriber. There are
three options:

1. no-verification - the state transitions directly to LOGGING_IN
2. fetch-attributes - attributes are fetched then transitions to LOGGING_IN
3. verify-email - the attributes are fetched and email_verified checked, then 
   transitions to LOGGING_IN or EMAIL_VERIFICATION_REQUIRED.

From LOGGING_IN default behaviour is to get a session (using `CognitoUser.getSession`) from which we can get an auth token, then establishing a session with a Federated Identity Pool.

If all of that succeeds we transition to 'LOGGED_IN'.
