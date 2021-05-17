# Software Development

## Developer
![developer](https://i.gifer.com/3Eqa.gif)

Nobody wants to ship software with lots of bugs, performance issues, and low customer satisfaction. Continuous integration and code reviews help prevent this... but who has the time, right? Well, agile teams maketime.

Agile developers focus on sustainable development–not heroics. Sustainability is about good estimation, effective branching strategies for managing code, automated testing to protect quality, and continuous deployment to get fast feedback from users. Adopting sustainable development practices requires a discipline most of us aspire to–but often struggle to realize–as individuals. That's because nobody can go agile in a vacuum. The culture of the entire organization has to rally behind it. That means getting project leaders to buy into the notion that quality is more important than scope or schedule, which is often the hardest part of adopting agile.

## Git

![git](https://memegenerator.net/img/instances/76792338.jpg)

### Tip 1: Start thinking about tasks as Git branches
At Atlassian, we create a new branch for every single issue. Whether it's a new feature, a bug fix, or a small improvement to some existing code, every code change gets its own branch.

Branching is straightforwards and allows teams to easily collaborate inside one central codebase. When a developer create a branch, they effectively have their own isolated version of the codebase in which to make changes.

### Tip 2: Multiple branches are individually testable, so take advantage
Once branches are considered done and ready for code reviews, Git plays another key role in an agile development workflow: testing. Successful agile teams practice code reviews and setup automated tests (Continuous Integration or Continuous Development). To help with code reviews and testing, developers can easily notify the rest of their team that the branch work is ready for review and that it needs to be reviewed through a pull request. More simply put, a pull request is a way to ask another developer to merge one of your branches into the master branch and that it is ready for testing.

With the right tooling, your Continuous Integration server can build and test your pull requests before you merge them. This gives you confidence that your merge won't cause problems. 

> **Pro Tip:**
> 
>A long running feature branch that is not merged to the master branch may hurt your ability to be agile and iterate. If you have a long running feature branch remember that you effectively have two divergent versions of your code base, which will result is more bug fixes and conflicts. But the best solution is to have short-lived feature branches. This can be achieved through decomposing user stories into smaller tasks, careful sprint planning, and merging code early to ship as dark features.

### Tip 3: Git provides transparency and quality to agile development
Adopting a regular release cadence is key to agile development. In order to make Git work for your agile workflow, you need to make sure that your master is always green. This means that if a feature isn’t ready then wait for the next release. If you practice shorter release cycles this shouldn’t and won’t be a big deal.

## Branching

![Branching](https://wac-cdn.atlassian.com/dam/jcr:a2441d68-3af8-4001-82c9-15c7b4b03799/GitBranchingDetailed.svg?cdnVersion=kr)

### Three branching strategies for agile teams
Branching models often differ between teams, and are the subject of much debate in the software community. One big theme is how much work should remain in a branch before getting merged back into master. 

#### 1. Release branching
Release branching refers to the idea that a release is contained entirely within a branch. This means that late in the development cycle, the release manager will create a branch from the master (e.g., “1.1 development branch”). All changes for the 1.1 release need to be applied twice: once to the 1.1 branch and then to the master code line. Working with two branches is extra work for the team and it's easy to forget to merge to both branches. Release branches can be unwieldy and hard to manage as many people are working on the same branch. We’ve all felt the pain of having to merge many different changes on one single branch. If you must do a release branch, create the branch as close to the actual release as possible. 

#### 2. Feature branching
Feature branches are often coupled with feature flags–"toggles" that enable or disable a feature within the product. That makes it easy to deploy code into master and control when the feature is activated, making it easy to initially deploy the code well before the feature is exposed to end-users.

#### 3. Task branching
At Atlassian, we focus on a branch-per-task workflow. Every organization has a natural way to break down work in individual tasks inside of an issue tracker, like Jira Software. Issues then becomes the team's central point of contact for that piece of work. Task branching, also known as issue branching, directly connects those issues with the source code. Each issue is implemented on its own branch with the issue key included in the branch name. It’s easy to see which code implements which issue: just look for the issue key in the branch name. With that level of transparency, it's easier to apply specific changes to master or any longer running legacy release branch.

### The merge
Branches tend to be short-lived, making them easier to merge and more flexible across the code base. Between the ability to frequently and automatically merge branches as part of continuous integration (CI), and the fact that short-lived branches simply contain fewer changes, "merge hell" becomes is a thing of the past for teams using Git and Mercurial.

That's what makes task branching so awesome! 

## Code reviews
![code_review](https://cdn-images-1.medium.com/max/1020/1*fAOWdn5tCzCqez6XMdWeVg.png)
### So, what exactly is a code review?
When a developer is finished working on an issue, another developer looks over the code and considers questions like:

- Are there any obvious logic errors in the code? 
- Looking at the requirements, are all cases fully implemented?
- Are the new automated tests sufficient for the new code? Do existing automated tests need to be rewritten to account for changes in the code?
- Does the new code conform to existing style guidelines?

### What's in it for an agile team?
#### Code reviews share knowledge
At the heart of all agile teams is unbeatable flexibility: an ability to take work off the backlog and begin execution by all team members. As a result, teams are better able to swarm around new work because no one is the "critical path." Full stack engineers can tackle front-end work as well as server-side work.

#### Code reviews make for better estimates
Estimation is a team exercise, and the team makes better estimates as product knowledge is spread across the team. As new features are added to the existing code, the original developer can provide good feedback and estimation. In addition, any code reviewer is also exposed to the complexity, known issues, and concerns of that area of the code base. 

#### Code reviews enable time off
Nobody likes to be the sole point of contact on a piece of code. Likewise, nobody wants to dive into a critical piece of code they didn’t write–especially during a production emergency. Code reviews share knowledge across the team so that any team member can take up the reins and continue steering the ship. (We love mixed metaphors at Atlassian!) Freedom to take that needed vacation, or freedom to spend some time working on a different area of the product.

#### Code reviews mentor newer engineers
A special aspect of agile is that when new members join the team more seasoned engineers mentor the newer members. And code review helps facilitate conversations about the code base. Often, teams have hidden knowledge within the code that surfaces during code review. Newer members, with fresh eyes, discover gnarly, time-plauged areas of the code base that need a new perspective. So, code review also helps ensure new insight is tempered with existing knowledge.

### But code reviews take time!
Sure, they take time. But that time isn't wasted–far from it.

***When done right, code reviews actually save time in the long run.***

Here are three ways to optimize for that. 

#### Share the load
At Atlassian, many teams require two reviews of any code before it's checked into the code base. Sound like a lot of overhead? Really, it's not. When an author selects reviewers, they cast a wide net across the team. Any two engineers can give input. This decentralizes the process so that no one is a bottleneck, and ensures good coverage for code review across the team.

#### Use peer pressure to your advantage
When developers know their code will be reviewed by a teammate, they make an extra effort to ensure that all tests are passing and the code is as well-designed as they can make it so the review will go smoothly. That mindfulness also tends to make the coding process itself go smoother and, ultimately, faster.

## Testing

![testing](https://www.ca.com/content/dam/ca/us/images/products/excuse-free-testing/no-time-for-complete-testing-sm.jpg)

### Moving from traditional to agile testing methods
Much like compounding credit card debt, it starts with a small amount of pain, but snowballs quickly–and saps the team of critical agility. To combat snowballing technical debt, at Atlassian we empower (nay: expect) our developers to be great champions for quality. We believe that developers bring key skills that help drive quality into the product:

- Developers are great at solving problems with code.
- Developers that write their own tests are more vested in fixing them when they fail.
- Developers who understand the feature requirements and their testing implications generally write better code.

We believe each user story in the backlog requires both feature code and automated test code. Although some teams assign the developers the feature code while the test team takes on automated testing, we find it's more effective to have a single engineer deliver the complete set.

> **Pro Tip:**
>
>Treat bugs in new features and regressions in existing features differently. If a bug surfaces during development, take the time to understand the mistake, fix it, and move on. If a regression appears (i.e., something worked before but doesn't anymore), then it's likely to reappear. Create an automated test to protect against that regression in the future.

### Human touch through exploratory testing
***Exploratory testing makes the code, and the team, stronger.***

But isn't exploratory testing manual testing? Nope. At least not in the same sense as manual regression testing. Exploratory testing is a risk-based, critical thinking approach to testing that enables the person testing to use their knowledge of risks, implementation details, and the customers' needs. Knowing these things earlier in the testing process allows the developer or QA engineer to find issues rapidly and comprehensively, without the need for scripted test cases, detailed test plans, or requirements. We find it's much more effective than traditional manual testing, because we can take insights from exploratory testing sessions back to the original code and automated tests. Exploratory testing also teaches us about the experience of using the feature in a way that scripted testing doesn't.

[what is exploratory testing ?](https://www.google.com/search?q=what+is+Exploratory+testing&oq=what+is+Exploratory+testing&aqs=chrome..69i57j0l5.4958j0j7&sourceid=chrome&ie=UTF-8)

## Continuous integration (CI)

![CI](https://cdn-images-1.medium.com/max/893/1*HP0Qss6BAQcv0UbHb21YFQ.png)

### What is continuous integration?
Continuous integration is the practice of routinely integrating code changes into the main branch of a repository, and testing the changes, as early and often as possible. Ideally, developers will integrate their code daily, if not multiple times a day.

### Protect quality with continuous builds and test automation
Two practices keep us out of that situation:

- **Continuous builds:** Building the project as soon as a change is made. Ideally, the delta between each build is a single change-set.

- **Test automation:** Programatic validation of the software to ensure quality. Tests can initiate actions in the software from the UI (more on that in a moment), or from within the backend services layer.

### Testing in CI: Unit, API, and functional tests
CI runs have two major phases. Step one makes sure the code compiles. (Or, in the case of interpreted languages, simply pulls all the pieces together.) Step two ensures the code works as designed. The surest way to do this is with a series of automated tests that validate all levels of the product. 

#### Unit Tests
Unit tests run very close to core components in the code. They are the first line of defense in ensuring quality.

- **Benefits:** 

    Easy to write, run fast, closely model the architecture of the code base.

- **Drawbacks:** 

    Unit tests only validate core components of software; they don't reflect user workflows which often involve several components working together.

#### API tests
Good software is modular, which allows for clearer separation of work across several applications. APIs are the end points where different modules communicate with one another, and API tests validate them by making calls from one module to another.

- **Benefits:** 

    Generally easy to write, run fast, and can easily model how applications will interact with one another.

- **Drawbacks:** 

    In simple areas of the code, API tests can mimic some unit tests.

Since APIs are the interfaces between parts of the application, they are especially useful when preparing for a release. Once a release candidate build passes all it's API tests, the team can be much more confident shipping it to customers. 

#### Functional tests
Functional tests work over larger areas of the code base and model user workflows. In web applications, for example, HTTPUnit and Selenium directly interact with the user interface to test the product.

- **Benefits:** 

    More likely to find bugs because they mimic user actions and test the interoperability of multiple components.

- **Drawbacks:** 

    Slower than unit tests, and sometimes report false negatives because of network latency or a momentary outage somewhere in the technology stack.




