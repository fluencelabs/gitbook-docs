# Adding A Storage Service

So far, all of the modules we have used were stateless and we did not have to give security much thought. If application A and application B both use our curl service, we don't run into any kind of conflict other than maybe scheduling. Not so for stateful services like a database. If we take no precautions with respect to authorization, other users of the service may, for example, alter or delete our data. And that's definitely not what we want especially if we plan on monetizing that data.

Without diving too deep into the Fluence security framework, you should be aware that Fluence has an out-of-the box authentication primitive that allows a `user == owner` check not unlike what we've seen in various blockchain platforms or plain old _sudo_. Of course, the Fluence security framework goes much further and affords developers a great deal of flexibility to code a security solution to their needs. For the purpose of our tutorial, however, we'll stick with the built-in authentication and use it as a base for [ambient authority](https://en.wikipedia.org/wiki/Ambient_authority). That is, we use authentication as an authorization guard for select functions at development time and provide the necessary credentials at service call time.

For the purposes of this tutorial, there is a caveat you need to keep in mind: Every reader of this document inevitably ends up using the same sample service with the same ownership control. In the highly, highly unlikely event you're getting funky results, it's most likely due to someone else doing the very same tutorial at the very same time. \[Jinx\]\([https://en.wikipedia.org/wiki/Jinx\_\(game](https://en.wikipedia.org/wiki/Jinx_%28game)\)\) ! Buy me a Coke, drink the Coke, slowly, try again and you should be fine.

The next sections explore both the setup and the use of our database: Sqlite as a Service.

