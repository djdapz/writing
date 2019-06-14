### outline





## Themes

### Builds don't always go as expected. You want to deploy the artifact that you vetted.

    - If you build twice...
        - No matter how good your CI tool is, it's not possible to guarantee that build environments are the exact same
                - Easier with concourse
                - Really hard with Jenkins
            - Caches from pevious or other application's builds
            - Something getting corrupted, a bit flipped the wrong way
            - Different versions of a build tool (gradle or npm)
        
        - You can't guarantee you'll get the same dependencies.
            - package.lock.json files DO NOT recursively specify every dependencies transitive dependencies.
            - pacakge.lock.json files can use "greater than" versioning rather than absolute
                - you could pull diffrent versions if you build at different times
            - WHY IS THIS BAD?
                - things may have unexpected interacitons
                - If you have a security dependcy tool, you can't rely on a scan early in the pipeline. you ahve sto scan each time.
                    - if something  bad is found, there's nothing you can do 
                    - Cant "shift left"
                -  dependenceis go away!!!! [Left Pad](https://www.theregister.co.uk/2016/03/23/npm_left_pad_chaos/)
        


### Artifact vs Source Code
    - Analogies
        - Cake vs Recipe -> Deployment analogy kinda falls apart
            - Environment can fuck you up
                - altitude
            - dependcies can fuck you up
                - type or granularity of flour
                
        - Legos vs build model
            - you don't twant to assemble the lego stucure each time
            - people loose pieces
            - people can assemble it wrong
            - stuff goes wrong
            - build the lego set once, make sure it looks good, then treat that as your product
            
### Artifact promotion

    - Confidence in deployment from each environment
    - You ran automted tests against THAT ARTIFACT in the pipeline
    - Yon ran autometed tests against THAT ARTIFACT in a dev environment
    - Your PM accepted the THAT ARTIFACT in staging
    - Your team saw THAT ARTIFACT integrate with dependencies in staging
    - You can now have confidence that THAT ARTIFACT will work well in prod.
    
    - This treats a build as a single unit. This build has been 'stamped' with various levels of approval as it flows through the pipelne
    - This is tracakble and lets you check the box 
    - Reduces complexity
    

          

### Enforces virtuous DevOps practices
    -  you can't go straight to prod, that's the point
        - do the hard things more often
        - makes teams care pipeline
            - speed
            - streamlined
            - efficient
            - reliable
        - nobody can buck up and send an artifact straight to prod, it has do be tested and sent through acceptance.
            - De risks the prod push 
            - Avoids compounding issues
        - by having an artifact of libraries, you can roll back while fixing things if appropriate
        
    - 12 factor app
        - by having one application, you have to configure using the runtime environment, not by different builds
        - app is portable
        
##Risks and Benefits

### risks of NOT doing this
    - Random errors in build
        - Caches
        - Corrupt artifact
        - Unexpected shit happesn
    - Dependencies having changed
        - NPM left pad
    - Package.lock DOES not solve for this
        - it uses minimum versions
        - it doesn't recursively track every transitive dependency
    - Left Pad 
    - You are techincally deploying something that is not testest or scanned

### benfitst of doing this
    - You can deploy something that you've seen work in lower environments
    - Think about promoting a build, not instructions for a build
        - Lego analogy
        - Cooking Analogy?
            - Don't love this one just because food goes bad haha
    - You have an artifact that has various levels of sign-off, certification, or 'stamps'
        - as it elevats from environment to environment its more and more blessed
        - use a diagram here?
        - Tools like concourse keep track of this nicely
    - You can roll back way faster
        - cut the dependency on a build server, it's just deployment you have to worry about 
    - pipelines faster
    - deliver more frequently
    - you can more reliably reproduce production errors
        - this allows you to run the exact piece of wfotware thats running in prod!
    
     
        
 