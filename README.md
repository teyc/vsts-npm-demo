# How to speed up Javascript builds on hosted VSTS agents

## Synopsis

`npm install` can take somewhere between 8-10 minutes when there are a lot 
of packages to be installed. I suspect `npm install` is CPU bound and network bound.

You can get around this by checking in your `node_modules` directory. However
this can make your repository clunky.

## Solution

Create another repository and commit your `node_modules`, then use `git submodule`
to link these two repositories together.

 [github](https://github.com/teyc/vsts-npm-demo-submodule)

1. Make sure you are using a 64bit version of nodejs.exe:

        $ node -p process.arch
        x64

2. Create a secondary git repository For example https://github.com/teyc/vsts-npm-demo-submodule

3. Copy the `package.json` and `package-lock.json` from the primary repository

4. `npm install` to restore packages

5. In the primary repository, add a submodule reference

       git submodule add https://github.com/teyc/vsts-npm-demo-submodule.git vsts-npm-demo-submodule

6. In VSTS, enable submodule to be checked out

   // TODO picture here

7. Replace the `npm install` step with a Powershell step to move the directory

       Move-Item vsts-npm-demo-submodule/node_modules .
       
## Potential issues

### node-sass

`node-sass` can be very stubborn even when package-lock.json is used. It uses a different
version of `binding.node` depending on CPU architecture and version of nodejs that is running
on the build server. 

You may have to commit every version of `node_modules/node-sass/vendor/win32-x64-??/binding.node` in order for your build
to work. See: https://github.com/sass/node-sass/releases

## Tips and tricks

1. Save a draft copy of the VSTS build definition, and work off there to avoid
   disrupting normal builds

2. Disable irrelevant build steps while you are testing

3. Git submodules refer to a particular commit. If you check in new node modules
   you must update the git submodule hash in your primary repository.

       git submodule update --remote



