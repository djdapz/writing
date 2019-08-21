# Goals of this post
1) Make this **interesting** for technical people
1) Make this **accessible** for non-technical people  
    - Communicate difference between source code and an article
1) Communicate value of only building once
    - de-risk deployments
    - speed up pipeline
    - provide confidence
1) Explain the idea of Artifact Promotion, why it's valuable, and why bulding once allows us to achieve it


# Iteration 3 -> Attempt to actually write (LOL)

### What's an artifact? Why should I care?


When a team deploys their code, they need to build it into an artifact first. 
The source code that teams write is not something that we can just run or deploy.
 
Almost all software project have a **build** process step where we do two main things.

##### Gather Dependencies
We gather all of the dependencies that our code relies on (other code that our code uses). These are often libraries React, the Spring Framework, or Django. Note that these dependencies almost always have dependencies of their own, we gather those as well.
##### Build the Artifact

We convert all of the code we need from the **human-readable** code that we write and maintain, and convert it into **machine-readable** code that will actually run on a comptuter. Many compilers or build tools will also have optimiztaion steps built in to make your code as efficient as possible.

For modern frontend JavaScript projects, it's a common practice to build the code into a single file called `bundle.js`, or a series of a few bundle files. Even though the browser can technically interpret our source, we get many advantages from the build step.
 - There are usually hundreds of dependencies and dozens to hundreds of .js files in a given project. Requiring a web browser to load each of these files independently would make page load times very slow. The build step gathers all the dependencies in the bundle so our browser doesn't need to make hundreds of requests to run the app. This dramatically improves page load times.
 - It's even better if we can remove un-necessary code. Many build tools preform [tree shaking](https://medium.com/@netxm/what-is-tree-shaking-de7c6be5cadd) to eliminate code from dependencies that our app doesn't actually use. The fewer lines of code we ship, the quicker the page will load. 
 - We use can automated tooling to make sure it's compatible across browsers. 
 - We can even make the code as sort as possible by renaming variables from things like `userPreferences` to `a`.
 - The build process can include a [transpliation](https://scotch.io/tutorials/javascript-transpilers-what-they-are-why-we-need-them) step. This enables us to use other languages like TypeScript, CoffeScript, Elm, Jsx, and still produce a JavaScript app that browsers can understand.
   

In a compiled language(Java, C++, Go ect.), the code is compiled into machine code. 
Machines cannot actually execute the code itself, they need a very detailed set of step-by-step instructions. 
The compilation process enables developers to work in an easily maintainable language that is closer to english and the business problems that they're trying to solve.
The compilation process also checks our code for us, it makes sure that we don't try to do things like subtract 15 from the word "potato". 


Here's an example of Some Java Sourcecode... [source](https://en.wikibooks.org/wiki/Java_Programming/Byte_Code)
```
 for (int i = 2; i < 1000; i++) {
  for (int j = 2; j < i; j++) {
    if (i % j == 0)
      continue outer;
  }
  System.out.println (i);
 }
 
 ```
 ...and it's respective Java ByteCode
 ```
  0:   iconst_2
  1:   istore_1
  2:   iload_1
  3:   sipush  1000
  6:   if_icmpge       44
  9:   iconst_2
  10:  istore_2
  11:  iload_2
  12:  iload_1
  13:  if_icmpge       31
  16:  iload_1
  17:  iload_2
  18:  irem             # remainder
  19:  ifne    25
  22:  goto    38
  25:  iinc    2, 1
  28:  goto    11
  31:  getstatic       #84; //Field java/lang/System.out:Ljava/io/PrintStream;
  34:  iload_1
  35:  invokevirtual   #85; //Method java/io/PrintStream.println:(I)V
  38:  iinc    1, 1
  41:  goto    2
  44:  return   
```

Regardless of the language, once these steps complete successfully we have an **artifact** that we can run or deploy. In the frontend JS world, we'd have a bundle.js, in Java we'd have .jar file, for C we'd have a .exe. 

### Cool so what?

![pipeline-img](build-many-pipeline.png)

Many teams use CI/CD tools such as Concourse, Jenkins, CircleCI, TeamCity to test their code, orchestrate their builds, and then deploy their software to various environments (dev, acceptance, staging, prod). 
When you want to deploy a specific version, it's very easy to simply reference a git commit. 
The tool will pull the code, create an artifact, then push the code to an environment. A super simple deploy step would look something like this;

```
    cd my-project
    cf target -s acceptance
    git check-out ${SHA}
    --env.ENVIRONMENT=acceptance npm run build 
    cf push -p build
```

#### What's the problem with this? 

The build process enables us to work in understandable high level languages and automates many optimization steps. It also introduces a lot of complexity and can be a point of failure.  

There's a lot that goes into it and a lot that can go wrong.

When a team builds before each deployment as in the example above, the deployed artifact is **un-tested**. 
It's like a chef serving a bowl of soup without tasting it because they know the recipe is good.

Does this REALLY matter though? Aren't computers really good at doing the same thing repeatably? Don't we have solutions like npm's `package.lock.json` or gradle's `build.gradle` to make sure builds are consistent?

Builds can always go wrong, and you really don't want your production build to be the one that has a problem.

- No matter what CI tool you're using, you can't guarantee that the build environment is the exact same. Many CI tools hold hold state across builds.
- Something can just go wrong. There's no such thing as bug-free software. Even compilers and build tools can have issues. 
- Dependencies can change can also change between builds. Since the artifact you deploy needs to incorporate its dependencies, you can end up deploying entirely different versions of dependencies!
    - package.lock.json files do not keep track of absolute versions, but rather minimum versions.
    - So, you could get newer versions of transitive dependencies baked into your artifact across different deployments.
    - Newer versions could have compatibility problems or security issues.
    - This makes it impossible to get reproducible and consistent builds.
- Dependencies can also go away over time. [Left Pad](https://www.theregister.co.uk/2016/03/23/npm_left_pad_chaos/) is an infamous example of what happens when a dependency gets taken down. The maintainers of Left Pad removed the library from NPM. Anyone with Left Pad as a dependency was unable to build their app, because a required dependency no longer existed.

We use environments for a reason, to de-risk going to the next one. Each acts a shield to the environment above it. This is how we can achieve a stable production environment while allowing developers to quickly iterate. If we build on the fly each time we deploy, we loose much of the shielding provided by lower level environments.


#### What's should we do instead?

![pipeline-img](build-once-pipeline.jpg)

Build your artifact once. Version it. Store it in an artifact repository. Promote it across different environments. 

You will have the confidence as you promote the artifact to higher environments. You will know that you're deploing the actual artifact, the exact sequence of 1's and 0's, that you vetted in lower environments. When you push that artifact to production, you know it's the EXACT same thing your PM accepted. It's the EXACT same thing that your tests passed against. It's the EXACT same thing that integrated with other apps and services in staging.

This will force you to follow factor [3 of 12 factor apps](https://12factor.net/config), externalize your environment specific configuration.

You will have the flexibility to deploy your artifact to any environment. You can roll back to older versions on demand.

Deployments will be fast. You don't have to waste time downloading dependencies and building each time.

There is less complexity in you DevOps setup. Fewer reasons for the pipeline to break.

