# Timid Technical Advice

The point of this doc is to share some advice to people starting a large serious application development process. Ultimately it is advice that I wish I had taken/known about when I started. 

As I’ll state at the end of this document, this advice is not for everyone and for every project (and I'm even sure there are those out there with better advice). Some projects will be technical so different, or will be small/temporary enough to avoid the below. But I’d wager that too many people would misclassify their project as smaller than it will really be (I certainly did). So keep that in mind reading the below.

Okay, without further ado: what is the technical advice I wish I had taken when starting development on a serious application?

## Linting

Linting ensures that code is of consistent style and can sometimes catch some bugs. While this might not seem powerful at first, consistency has a lot of benefits. You will be able to jump into code in different parts of the application written by different people and you will still be familiar with the way the code is written.

There are not many rules when it comes to linting, but I will offer two:

* Configure linting early
* Don’t allow merging unlinted code

These two are pretty self explanatory. 

If you wait some time to configure it for your team, your code will probably drift and it will be a huge pain pulling all code back to one consistent style (a single PR hitting 100 files or the like).

The fact that you should not allow unlinted code to be merged in is pretty simple to understand as well. You might even want to set up a precommit hook for it. But the least you should do is prevent merges. It is easy to fix and if you really need to write code that breaks the linter, you should be forced to go out of your way to get it in the repository.

Last bit of advice, pick a linter that many people use and is recommended for your codebase (vue will have different preferences than react). 

## Review 

Reviews are a great way of finding errors and maintaining consistency. Often another set of eyes will be able to find things that the first missed.

Again there is a pretty easy rule of thumb: always review. Nothing should be urgent enough that only one pair of eyes looks at it. 

When I do reviews, I do two parts: 1) a code review where I just check out the code and 2) a functionality review where I repeat the functionality covered in the issue. Do whatever you want, but be thorough and don’t be afraid to comment if you think it could be better.

## Testing

### How to test an application

Testing an application has obvious and hidden benefits. The obvious benefit is that you will be able to catch bugs and have fewer errors. The hidden benefit is that by testing your code, you often discover new and better ways to write it in the first place.

The problem is that unlike the previous two good practices, this one is more effort to implement. That being said I think it will save you time in the long run, and I would not have put this here if it were not worth it.

Here are three practices that I have used when it comes to tests:

1. Add a test for every new change
1. Add a test for every fixed bug (these are called regression tests)
1. Before pushing new changes to your users, run all tests (this is called continuous integration)

The first of these maxims means that your code will be pretty well tested. This will catch bugs before they happen in individual parts of the code. The second, regression tests, will make sure that you have fixed bugs that crop up. And in conjunction with continuous integration will prevent them from happening in the same way again. The third, continuous integration, will catch bugs that happen in complex interactions of the code. This final step is the most important one and will help catch bugs before they get to your users.

If there is one thing that you can take away from this, it is to use continuous integration, but if they are three things then remember the three above.

### Levels of tests

(This part is a bit theoretical and can be skipped without too much worry.)

The ideal test is: having billions of perfectly simulated fake users constantly interact with your site. This is of course pretty impractical, but it acts as a goal post for what tests should look like. Unfortunately the more tests deviate from this ideal, the easier the tests are to implement. We are constantly striving for a balance between high coverage/fidelity tests and velocity/rigidity. That being said, let’s dive into some types of tests that exist out there in practice (note others will have different definitions of the below words - but it is the definition that matters).

A system test is about as close to the ideal as is possible. A system test acts against the entire application, so it does not necessarily isolate a specific part of the application, but rather a specific flow. You will need to write such a test against an environment designed to support these tests, so without multiple environments (more on that in a bit), you probably won’t do system testing. These tests are also very hard to write. You will be simulating a user’s behavior, so not only will you need to interact with your application in a visual way, and there is often very high amounts of branching that happens at this level. All that being said, system tests are hard to maintain and hard to write. For anything but the most crucial applications I would avoid writing system testing without carefully crafted tools. There certainly are some tools out there that make some system testing easier but there's been none that comes without any hitch.

An integration test doesn't simulate a user. Instead it tests functions/modules inside of your code. While testing these modules, you’ll want to keep all other aspects of the application the same as the live application. In such a way you will be able to ensure that this is as close to the ideal test case as possible. The nice thing about these tests is that you are interacting with code, not a user interface. And because these tests generally work on more specific components, there will be less branching. 

There are problems with integration tests however. Because these tests will be actually calling APIs, there will be a bit of flakiness that you’ll get here because of connection issues. Some third party APIs also don’t support integration testing. For example, with the twilio API there's no way to send a test text, unlike with the stripe API. If you sent a request to twilio it will treat it like a real text and actually send a text and charge you. While integration testing can catch many issues, some are hard to catch. Issues with user interface or edge cases in user state can sometimes be overlooked. 

Unit tests are the final type of test. These tests will again test at the code level. The nice thing about them though is that they don’t require you run the rest of the application. Instead of calling other functions, you will stub/mock these functions (this means you will pretend that these functions executed and returned what you wanted). This makes it very easy to reason about and cuts the branching factor. This reduces flakiness and makes the tests very fast to run. Unfortunately, if you don’t mock correctly (the map is not the territory), then your tests will most likely not represent reality.

Ultimately integration testing is often the best middle ground. Especially if you were using serverless development, you will find simulating API requests to production looking environments much easier. 

Hopefully this section will guide you in the tradeoffs of various levels of tests, and give you some intuition on where to begin. We began with integration and unit tests, and only recently started running system tests on crucial user flows.

## Multienv

One powerful tool in SWE is having multiple versions of your application running at the same time. We refer to these different applications as different environments. Having multiple environments is a clever way of predicting the future and thus preventing bugs.

I'd recommend to start with (roughly) three versions of your application: development, staging and production. You can of course have more, but I’d start with these three. As soon as you have real users using your site you should have all three - so implement and think of these early. 

Production is where real user data lives. Have caution and be careful with having untested code operating on this environment. You should monitor this environment (more on that in a bit), not test it.

Staging should be a future version of prod without all the user data. It should act as your canary and QA. You should be able to see how your next changes will look and feel. You will find bugs here before you find them in production. The goal is that staging should be as close as you can get to a future version of your production environment. When you deploy from staging to production (we will talk about what this means in one second), in that instant their code should be identical.

We want the two environments to be so similar, because if they are not, then staging loses its meaning. The goal of staging is to be a signal of what is to come. If the two environments were dissimilar then tests on staging would have no bearing on how the production environment would do.

The final environment is dev. This environment should be used for development work and can be completely messed up. The key to a development environment is 1) not only can it be completely messed up without repercussions. But 2) it can also be pulled back from the edge. There should be some way to restore a development environment to some semblance of sanity through an easy process.

So these are the three envs and their ideal use cases. How do we achieve them?

Well there are ultimately three things that we need to worry about: code, config and data.

### Code 

The code is the product in a sense. And for the most part, this is what you are interested in future proofing. For that reason, you'll want the code in your prod environment to be the oldest and stablest code. The code in your staging environment will be the next future change you want to prod. And the code in dev is the code that will deploy far in the future.

Once again we see the environments core use cases shining. The production environment is the one that's most stable, the staging environment is going to be a prediction of how the production environment will be in the future, and the development environment will allow you to do anything that you would like.

We use a system of merging branches ensure that all of our development branches are merged first into staging, and then only after tests pass are merged into production.

### Configuration

Configuration is the second most important thing that you will need to share between these environments. What is configuration? 

Well if you were using the same code across different environments, then you run into some awkward things. Specifically what would you do about hitting a third party's production API or about processing payments. You probably don't want to test out processing payments on your staging environment with real money. So how do you configure the staging environment such that it doesn't accept real money and the production environment such that it does. This configuration is called the config.

In order to run multiple environments, you will need to use configuration files. So that means you will have a separate config for development staging and prod environments. You'll need to a) come up with a system to ensure that each environment gets its correct configuration, b) make sure that people know where these configurations are, and c) people can easily update them.

Depending on the structure of your application, and the tools that you choose to use, there are many ways to deal with config files. We use an automated way to deploy your configuration files to our specific environments. This process becomes more important as the business gets bigger, because the configuration files often contain private information.

We use Amazon Secrets manager to store configuration files and a continuous integration / deploy provider to store the key to Amazon Secrets manager and parcel out the configuration files to the correct environments. Of course when a developer first starts they will generally need to download some configuration so that their local environment will work for  development. We have enabled that as well.

### Data   

Dealing with data across multiple environments has its own problems. Generally speaking there are two types of data. The first type is data that is created by you. This might be a blog article or picture on your site. This type of data in some senses is very similar to code, because as you make new changes to your site you'll create new data and you'll want to test to see how the data looks and functions on a staging environment. Treating this data like code can often be a good way to start, however as your project and company grows you will probably need to build out custom solutions to deploy this data.

There's a second type of data and this data is user data. This is something that you won't create yourself but, users of your platform will create. When it comes to user data people generally fall into two camps (specifically when it comes to code development and not data science or Business Analytics). The first camp is that you should never use user data. If you need this data to test something in your application you should go ahead and create fake data in order to test. This is a principled approach, however fake data is never as realistic as real data and this is the critique that the other camp has of the first. The second camp uses real data in order to do tests. However they scrub this data for any personal identifying information. The problem here being that you can never completely scrub all PII from user data. 

I tend to fall into the first camp myself however I've not been on a project we're making fake data is incredibly hard. (Note that none of this is from a data scientist perspective)

### Multi environment conclusion

Having multiple environments is crucial in order to efficiently test future changes to your application: both for code and for content changes.

The three environments above can be a great start; however, as applications grow, finding bugs will get harder and harder (and often fall into extreme edge cases). Finding these bugs will require more and more people, thus an internal staging/QA env is often not enough. To compound this problem, some bugs are quite disastrous, and will need to be found quickly. To balance both of these concerns companies will generally have a staggered release cycle,  such that new features will slowly get pushed out to larger and larger audiences, where stability matters more and more.

The series of environments needed to effectively implement a staggered release cycle are complex. Even the initial three environments can be complex enough, and complexity can create errors. You should  strive to  automate your team's use of environments as early as possible. The sooner taking code from dev to staging to prod becomes as easy as a single button push, the better.

## Serverless

One thing you may have found to be conspicuously missing in the above description of multiple environments testing, is the hardware these environments are running on.

My suggestion, not a commandment, is that you consider using serverless architectures in order to begin your application development. 

While in many situations this is infeasible, if you are able to ignore the hardware and the environment that your code is running on and instead focus on code and application development you will generally have a faster time developing. 

So maybe the conclusion here is that if you can't think of a good reason to host it yourself let another party do so.

A final note is that most things can be hosted in a serverless way these days by relying on third parties to manage the underlying infra: email, text, databases, logs, GPUs for training NNs, etc.

## Structure

I want to talk a little bit about how to structure  an application (a little bit because I don’t know enough to say more).  A good rule of thumb is to reuse. Decisions and structures that reuse themes from other parts of your app will be better than those that don't.

One example is to reuse data structures on the front end and the back end. If you use a data structure on the backend consider using that same data structure on the front end as well to present and store that data. Another happens when naming: if a module on the front end does something that is exposed on the backend, considered naming it similarly. 

Names are one of the hardest things to get right (especially for me), you should you put some thought into them and commit early to conventions. Oftentimes conventions, even if not perfect, will be better than having no convention at all. So if you are building something and unsure what the naming convention should be, Google it, I'm sure someone out there has built something similar and developed a naming convention. One example of this is API structure, if you're not reinventing the way we communicate on the web just be restful.

There are plenty of other small tidbits that I could advise, such as I particularly like having tests with code bucketed in similar folders, but we would be getting into the weeds (and everyone has the right to choose what weeds they have in their garden). 

Overall my advice here is to find a convention, stick to it, and make sure it's well-documented so others will stick to it as well.

## Logging and monitoring 

You can't test everything. And because we live in a world where such a travesty is true you will need to have monitoring and logging (basically recording how the production environment is doing). 

In the case of logging just remember that storage is generally cheap. Be thoughtful with your logs and think with errors in mind. I cannot tell you how helpful it is to have great logs when you are trying to drill down on to a very elusive bug.

As for monitoring think of your business use cases. Monitoring is only useful if people check those monitoring logs.  So you will want to be very judicious about when you will throw an alert from monitoring. Throw alerts that are tied to business use cases such that monitoring/alerting will only wake people up if it's absolutely necessary.

For a very quick tidbit on monitoring and logging used for data science, see below.

## SQL and dashboards 

Finally just a bit on data science. When structuring your data, spare an instance of thought for how data scientist will use it in the future. Think how this data will fit into a SQL table (trying to be realistic here). Think about what your CEO might be interested in, and try to keep data that could construct a dashboard that would tell investors or your CEO the health of your business. 

## Conclusion

Take everything above with a grain of salt. Each maxim has it’s time and place, and there will be many of the above that don’t apply to you, at your company’s stage, for your technical challenge, etc. But hopefully there is some wisdom wrapped up somewhere in there.

Appreciate you reading this far, and constructive comments are more than welcome.
