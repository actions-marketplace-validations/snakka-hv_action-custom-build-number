# action-custom-build-number
Maintain custom build number for the git branches using GH actions

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - name: Generate build number
      uses:  snakka-hv/action-custom-build-number@master 
      with:
        token: ${{secrets.github_token}}        
    - name: Print new build number
      run: echo "Build number is $BUILD_NUMBER"
```

After that runs the subsequent steps in your job will have the environment variable `BUILD_NUMBER` available. If you prefer to be more explicit you can use the output of the step, like so:

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - name: Generate build number
      id: buildnumber
      uses: snakka-hv/action-custom-build-number@master 
      with:
        token: ${{secrets.github_token}}        
    
    # Now you can pass ${{ steps.buildnumber.outputs.build_number }} to the next steps.
    - name: Another step as an example
      uses: actions/hello-world-docker-action@v1
      with:
        who-to-greet: ${{ steps.buildnumber.outputs.build_number }}
```
The `GITHUB_TOKEN` environment variable is defined by GitHub already for you. See [virtual environments for GitHub actions](https://help.github.com/en/articles/virtual-environments-for-github-actions#github_token-secret) for more information.

## Setting the initial build number.

If you're moving from another build system, you might want to start from some specific number. The `build-number` action simply uses a special tag name to store the build number, `build-number-x`, so you can just create and push a tag with the number you want to start on. E.g. do

```
git tag build-number-200
git push origin build-number-200
```

and then your next build number will be 201. The action will always delete older refs that start with `build-number-`, e.g. when it runs and finds `build-number-200` it will create a new tag, `build-number-201` and then delete `build-number-200`.

## Generating multiple independent build numbers

Sometimes you may have more than one project to build in one repository. For example you may have a client and a server in the same github repository that you would like to generate independent build numbers for. Another example is you have two Dockerfiles in one repo and you'd like to version each of the built images with their own numbers.  
To do this, use the `prefix` key, like so:

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - name: Generate build number
      id: buildnumber
      uses: snakka-hv/action-custom-build-number@master
      with:
        token: ${{ secrets.github_token }}
        prefix: client
```

This will generate a git tag like `client-build-number-1`.

If you then do the same in another workflow and use `prefix: server` then you'll get a second build-number tag called `server-build-number-1`.

## Branches and build numbers

The build number generator is global, there's no concept of special build numbers for special branches unless handled manually with the `prefix` property. It's probably something you would just use on builds from your master branch. It's just one number that gets increased every time the action is run.

## Reference
TBA (Found the base script from an MIT project. To be linked still)