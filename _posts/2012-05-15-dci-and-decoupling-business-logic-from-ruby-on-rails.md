---
layout: post
title: DCI and decoupling business logic from ruby on rails
---

Every so often the rails community adopts new best practices and patterns because someone figured out a better way to architect their applications. One great example was when everyone was urged to push the logic out of controllers and move it into models. This solved a lot of issues for most applications. Now we're able easily isolate and test away the logic on the model layer instead of the hard to isolate controller layer.

Pushing the logic to the models created a nasty side effect however. Our models became completely bloated and hard to maintain. They contained logic that we tried to abstract away into model classes or new modules inside of the lib directory with one-off abstractions with no clear home. I'm sure some of us already knew there was something wrong, but perhaps it was only a gut feeling because no one piped up! It wasn't until I saw Uncle Bob's [keynote speach](http://www.confreaks.com/videos/759-rubymidwest2011-keynote-architecture-the-lost-years) at [Ruby Midwest 2011](http://www.rubymidwest.com/) that I realized there was a huge flaw in the status quo of rails development.

His keynote completely changed the way I code web applications and Ruby applications in general. He suggested that too much of the business logic of rails applications was locked up inside of the models and the controllers. Everytime he opened up a rails application, he saw a rails application and not an "invoicing application" or a "CRM". The most important part of the application seemed to be the framework upon which it was built. Rails is a web framework and is simply a detail or delivery mechanism to the web for your true (invoicing/crm/next social network) application.

He described how to get the intent of the application at the top level by adopting a use case driven approach to software engineering. It starts with the interactor object that implements the use case. It contains application specific business rules that interact with entities which have application independent business rules. Entities are not models and are decoupled from the data. The delivery mechanism calls our business layer which includes the interactors and entities. Since the business layer doesn't depend on the anything else, it can be packaged into its own gem and tested in isolation. He mentioned the concept of a boundary object, but in Ruby's case I think we can safely ignore them. You should really watch the video if you want to get the meat of his talk. He's also pretty entertaining. :)

During our work on our enterprise project called [SettlementApp](http://settlementapp.com), I think I've finally found a general architecture I am happy with that has significantly improved code quality and maintenance. It combines concepts from Uncle Bob and a fairly new concept called DCI into one.

Here is the break down of topics involved:

* [Decoupling business logic from the web application](/2012/05/15/decoupling-business-logic-from-the-web-application.html)
* Decoupling the infrastructure layer (**Coming soon!**)
* DCI (Data Context and Interactors) (**Coming soon!**)

I think we're on the verge of another great pivot in our thinking. Feedback is appreciated.

References:

* [Keynote: Architecture the Lost Years](http://www.confreaks.com/videos/759-rubymidwest2011-keynote-architecture-the-lost-years)
* [Object Oriented Software Engineering: A Use Case Driven Approach](http://en.wikipedia.org/wiki/Object-oriented_software_engineering)

Resources: 

* [Setting up your own private gem server](/2012/05/14/private-gem-server-using-geminabox.html).
* [James Coplien - The DCI Architecture: Supporting the Agile Agenda](http://vimeo.com/8235574)
* [Trygve Reenskaug - DCI: Re-thinking the foundations of object orientation and of programming](http://vimeo.com/8235394)
* [http://clean-ruby.com/](http://clean-ruby.com/)

[HN Discussion](http://news.ycombinator.com/item?id=3976806)