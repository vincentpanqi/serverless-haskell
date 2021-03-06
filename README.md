# Serverless Haskell

[![Build status](https://img.shields.io/travis/seek-oss/serverless-haskell.svg)](https://travis-ci.org/seek-oss/serverless-haskell)
[![Hackage](https://img.shields.io/hackage/v/serverless-haskell.svg)](https://hackage.haskell.org/package/serverless-haskell)
[![Hackage dependencies](https://img.shields.io/hackage-deps/v/serverless-haskell.svg)](https://packdeps.haskellers.com/feed?needle=serverless-haskell)
[![npm](https://img.shields.io/npm/v/serverless-haskell.svg)](https://www.npmjs.com/package/serverless-haskell)

Deploying Haskell code onto [AWS Lambda] using [Serverless].

## Requirements

* AWS account
* [Stack]
* [NPM]
* [Docker] unless running on a Linux host

## Usage

* Create a [Stack] package for your code:

  ```shell
  stack new mypackage
  ```

  LTS 9 and 10 are supported, older versions are likely to work too but untested.

* Initialise a Serverless project inside the Stack package directory and install
  the `serverless-haskell` plugin:

  ```shell
  cd mypackage
  npm init .
  npm install --save serverless serverless-haskell
  ```

* Create `serverless.yml` with the following contents:

  ```yaml
  provider:
    name: aws
    runtime: nodejs6.10

  functions:
    myfunc:
      handler: mypackage.myfunc
      # Here, mypackage is the Haskell package name and myfunc is the executable
      # name as defined in the Cabal file

  plugins:
    - serverless-haskell
  ```

* Write your `main` function:

  ```haskell
  import qualified Data.Aeson as Aeson

  import AWSLambda

  main = lambdaMain handler

  handler :: Aeson.Value -> IO [Int]
  handler evt = do
    putStrLn "This should go to logs"
    print evt
    pure [1, 2, 3]
  ```

* Use `sls deploy` to deploy the executable to AWS Lambda. **Note**: `sls deploy
  function` is not supported.

  The `serverless-haskell` plugin will build the package using Stack and upload
  it to AWS together with a JavaScript wrapper to pass the input and output
  from/to AWS Lambda.

  You can test the function and see the invocation results with `sls invoke
  myfunc`.

### Notes

* `sls deploy function` is not supported.
* Only AWS Lambda is supported at the moment. Other cloud providers would
  require different JavaScript wrappers to be implemented.

See
[AWSLambda](https://hackage.haskell.org/package/serverless-haskell/docs/AWSLambda.html)
for documentation, including additional options to control the deployment.

## Testing

* Haskell code is tested with Stack: `stack test`.
* JavaScript code is linted with `eslint`.

### Integration tests

Integration tests are not run automatically due to the need for an AWS account.
To run them manually:

* Ensure you have the required dependencies: `curl`, [jq], [NPM] and [Stack].
* Get an AWS account and add the access credentials into your shell environment.
* Run `./integration-test/run.sh`. The exit code indicates success.

## Releasing

* Run the integration tests.
* Install [bumpversion](https://github.com/peritus/bumpversion): `pip install bumpversion`.
* Run `bumpversion major|minor|patch`.
* Run `git push --tags && git push`.

[AWS Lambda]: https://aws.amazon.com/lambda/
[Docker]: https://www.docker.com/
[jq]: https://stedolan.github.io/jq/
[NPM]: https://www.npmjs.com/
[Serverless]: https://serverless.com/framework/
[Stack]: https://haskellstack.org
