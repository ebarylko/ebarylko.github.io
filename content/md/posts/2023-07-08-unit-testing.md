{:title "Unit testing"
 :layout :post
 :tags  ["cljs" "testing"]
 :toc true}


## Overview

This article covers what unit testing is, why it is useful, and an example of unit testing in a project.

## What is unit testing?

 Unit testing is about testing a function, using the inputs and verifying the outputs. Unlike acceptance testing which tests the result of a collection of functions interacting together, unit testing just grabs one function and sees whether it reacts correctly given certain inputs.
 

## Why is it useful?
 
One possible situation in where you could have a unit test is if you are testing the registration feature for a website. What could be tested is that given the information for a user, registering a user with that information should result in a new user appearing in the database. That user's information should match the same information provided when registering earlier.

This test allows you to prove that user registration for the website works. Furthermore, testing individual components allows problems to be diagnosed more quickly when debugging. Knowing which components work allows the faulty components to be identified and fixed.

## How do I use unit tests in tandem with accepttance testing?

With acceptance testing, you test that a collection of events occur in a specific result. If you want to test every result for all event combinations, you will end up with countless tests.

Using a combination of unit testing and acceptance testing, you can still test certain event combinations while assuring that other event combinations would not occur.

For example, If I had a website where the user sees their dashboard after logging in at the home page, I could have one acceptance test check that scenario. Given a user who heads to the home page and logs in with the correct credentials, they will then see their dashboard. To be confident that users who enter incorrect credentials don't see their dashboard, I would like to have some tests that can confirm this. However, repeating the earlier acceptance test with countless variations of incorrect credentials would be a lot of effort.

On the other hand, If I know that the user only sees their dashboard if they are authenticated, I could instead test my function which validates the user with a slew of incorrect credentials. Since viewing the dashboard depends on the result of the validation function, proving the function does not validate the user given incorrect credentials shows that an unauthenticated user cannot view their dashboard.


## A real life example

I will show an example of a unit test from a document sharing site I am working on. In this test, I check whether a registered user can login with hardcoded credentials and whether their account information is updated accordingly.

Here is the code for the test.

```clojure
(t/deftest registered-user-login
  (rf-test/run-test-async
   (routes/init-routes!)
   (core/firebase-init!)
   (rf/dispatch-sync [::events/initialize-db])
   (rf/dispatch [::events/login])
   (rf-test/wait-for [::events/fetch-documents] 
    (let [user (rf/subscribe [::subs/user])
          name (rf/subscribe [::subs/name])]
      (t/are [x y] (= x y)
        :registered @user
        "han@skywalker.com" @name)))))
```

First, I initialize my routes, setup the database, and connect to firebase. After that, I login and wait for the user's documents to be fetched. I then check whether the name of the user matches the credentials entered earlier and if the user is noted as registered.

This test confirms that this aspect of the application works. With this I can always check to see whether logging in works. The moment it fails, I can look around and fix the issue. This is better than having this go unnoticed, for if a user can access an account with incorrect credentials, other user's information can be tampered with and taken.


## Conclusion

Unit tests serve as a way of checking the inputs and outputs of functions, allowing for their correctness to be checked. Furthermore, they can also be used with acceptance tests to show that certain scenarios will not occur if those scenarios depend on the result of a specific function. 





