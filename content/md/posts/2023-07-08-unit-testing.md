{:title "Unit testing"
 :layout :post
 :tags  ["cljs" "testing"]
 :toc true}


## Overview

This article covers what unit testing is, why it is useful, and an example of unit testing in a project.

## What is unit testing?

 Unit testing is about testing a function, using the inputs and verifying the outputs. Unlike acceptance testing which tests the resullt of a collection of functions interacting together, unit testing just grabs one function and sees whether it reacts correctly given certain inputs.
 
## Why is it useful?
 
One possible situation in where you could have a unit test is if you are testing the registration feature for a website. What could be tested is that given the information for a user, after registering a user with that information should appear in the database for users.

This test allows you to prove that user registration for the website works. Furthermore, testing individual components allows problems to be diagnosed more quickly when debugging. Knowing which components work allows the faulty components to be isolated and fixed.

## How do I use unit tests in tandem with accepttance testing?

With acceptance testing, you test that a collection of events occur in a specific result. If you want to test every result for all event combinations, you will end up with countless tests.

Using a combination of unit testing and acceptance testing, you can still test certain event combinations while assuring that other event combinations would not occur.

For example, If I had a website where the user sees their dashboard after logging in at the home page, I could have one acceptance test check that scenario. Given a user who heads to the home page and logs in with the correct credentials, they should then see their dashboard. To be confident that users who enter incorrect credentials don't see their dashboard, I would like to have some tests that can confirm this. However, repeating the earlier acceptance test with countless variations of incorrect credentials would be labourous. 

On the other hand, If I know that the user only sees their dashboard if they are authenticated, I could instead test my function which validates the user with a slew of incorrect credentials 








