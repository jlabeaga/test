# README #

### How do I get set up? ###

To have this project up and running you will need:
* Java 
* Maven
* Node (>12)
* AWS SAM CLI

# Install

Execute this commant in your root folder

```shell script
npm install
```

### Start local api ###

In order to start the api, there's a bash script to help you achieve it.

```shell script
./start-api.sh service-name
```

### How to create a new service ###

From the root of the project, execute this command:
#### Serverless ####
```shell script
serverless create --template aws-java-maven --name service-name --path service-name
```

This will generate a base app with all set up. To adapt to our needs, several things needs to be changed:

Change runtime to java 11 and import the environment variables from serverless.common.yml

```yaml
# serverless.yml
provider:
  # provider stuff
  runtime: java11
  environment:
    ${file(../serverless.common.yml):environment}
```

Environment variables affecting all projects can be defined in serverless.common.yml.

Environment variables affecting one project can be defined in its own serverless file (same for function)

Change the default package artifact to the current directory

````yaml
package:
    artifact: ./
````

Add the serverless sam plugin to the serverless.yml.

````yaml
# serverless.yml
plugins:
    - serverless-sam
````

#### NPM ####

Generate a package.json with two scripts inside the service folder:
```json
{
  // package.json stuff
    "scripts": {
      "export-sn": "node ../node_modules/serverless/bin/serverless.js sam export --output template.yaml",
      "start-sn": "sam.cmd build && sam.cmd local start-api -p PORT"   
    }
}
```
"-sn" stands for "service-name".
* export-sn will generate the sam template to execute our lambdas
* start-sn will build all functions and start a simple local api server in the specified port. By default 3000.

### How to add new Lambda to the CI

1. In the Lambda's sub-folder, create a new file named `lambda.config`:
```yaml
# Name of the Lambda function.
name = "aws-java-simple-http-endpoint"
# URI of API endpoint to use (example: "v1/my-lambda/some-path").  
path_part = "simple-http-endpoint"
# HTTP methods, which Lambda supports.
http_methods = ["DELETE", "GET"]
# Lambda's handler.
handler = "com.serverless.Handler"
# Lambda's runtime.
runtime = "java11"
# Binary file which will be deployed (specified in Maven's configuration).
src_file = "target/aws-java-simple-http-endpoint.jar"
```

2. Merge the branch with new Lambda to "develop" / "stage" branch. CI will compile and deploy Lambda(s).

### CI process

On a merge to develop or stage branch (@see `bitbucket-pipelines.yml`), CI:
1. Installs dependencies (@see `.infrastructure/scripts/ci/00-init`);
2. Compiles Lambdas (@see `.infrastructure/scripts/ci/10-build-lambdas`);
3. Deploys Lambda using Terraform (@see `.infrastructure/scripts/ci/20-deploy-lambdas`).
