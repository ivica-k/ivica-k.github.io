---
title: "The end"
chapter: true
weight: 90
---

# This took a lot of effort! 
## Great job everyone and thank you for being here <i class="fas fa-heart"></i>

The application we built today can of course be improved :) That does not mean that we didn't do an amazing job!

Take some time to reflect on the amount of work and learning accomplished today.

Getting here has been a long time coming. Donors, donations, tables, keys, secondary indexes with inverted keys, 
streams, permissions, deployments...

We have done _so much_ in this workshop, and you should be proud of that!

#### Cleanup
If you would like to make my life a bit easier, execute `chalice delete` to clean-up the resources you created today.

## Where to go from here?

One of the improvements we could make to the overall application is to build the infrastructure using
[AWS CDK for Python](https://docs.aws.amazon.com/cdk/v2/guide/home.html) instead of using the AWS CLI. This would ensure 
that the application infrastructure is repeatable, can be code-reviewed and rolled back if necessary.

Authentication and authorization is a topic we did not touch upon but Chalice [does support several authorizers](https://aws.github.io/chalice/topics/authorizers.html).
The application as we built it is completely open and anyone can invoke the API and incur cost for us :/

More tests and more error handling would be beneficial.

## Feedback and socials

I would _greatly appreciate_ any feedback you may have about this workshop. [Feedback form](https://forms.gle/J5tkKYpqaSkE6vA4A).

Let's connect!
 - <a href="https://www.linkedin.com/in/ivica-kolenka%C5%A1-8b333895/" target="_blank">LinkedIn <i class="fab fa-linkedin"></i></a>
 - <a href="https://www.twitter.com/nekybrate" target="_blank">Twitter <i class="fab fa-twitter"></i></a>
 - <a href="https://www.github.com/ivica-k" target="_blank">Github <i class="fab fa-github"></i></a>