# RudderStack Transformation Action

This Github Action allows developers to test and publish user transformations and custom libraries directly from their development repository. To know more about user transformations check [here](https://rudderstack.com/docs/transformations).

## Prerequisites

- You will need the email address associated with your RudderStack workspace.
- Generate a [Service Access Token](https://www.rudderstack.com/docs/dashboard-guides/service-access-tokens/) with [Admin](http://www.rudderstack.com/docs/dashboard-guides/user-management/#organization-roles) permissions.

<img width="465" alt="sat-permissions-action" src="https://github.com/user-attachments/assets/d0b8b937-28ba-423e-b7e2-de4a4c70752d" />

Note that:

- For production use cases, RudderStack recommends using a Service Access Token instead of [Personal Access Token](https://www.rudderstack.com/docs/dashboard-guides/personal-access-token).
- For security purposes, it is highly recommended to use [GitHub secrets](https://docs.github.com/en/actions/security-guides/encrypted-secrets#creating-encrypted-secrets-for-a-repository) to store your Service Access Token.

## Usage

```yaml
name: Rudder Transformer Test and Publish
uses: rudderlabs/rudder-transformation-action@<version>
with:
    metaPath: './code/meta.json'
    email: 'test@rudderlabs.com'
    accessToken: ${{ secrets.ACCESS_TOKEN }}
    uploadTestArtifact: true
```

> Note: The action does the work of testing a transformation for a given set of events, it's creation and updation along with any custom libraries using the [transformation API](https://www.rudderstack.com/docs/api/transformation-api/). Read more in [this blog post](https://rudderstack.com/blog/rudderstacks-transformations-api). 

> For the action to work, one would need the workspace email and accessToken. Learn how to generate accessToken [here](https://www.rudderstack.com/docs/dashboard-guides/personal-access-token/).

> For examples of using the action, checkout this [sample repository](https://github.com/rudderlabs/rudder-transformation-action-code/tree/main/.github/workflows).

> We recommend using git secrets to store your accessToken for security purposes and use it as mentioned in the above example

## Action Spec

### Inputs

- `email` (required) : RudderStack app workspace email.
- `accessToken` (required) : RudderStack app corresponding accessToken.
- `uploadTestArtifact` (optional) : boolean flag on whether to upload the individual transformation outputs after running the  transformation on the test events and it's diff from expected output for each.
	- When test-input-file is provided, actual outputs of all transformations with respective inputs from test-input-file are dumped into artifacts
	- When expected-output is provided, the above outputs are validated against the contents in expected-output and a diff is returned in artifacts if there is any.
	- Transformation outputs of the test data is written in its respective `camelCase(Name)_output` file
- `metaPath` (required) : The path to the meta file, the meta file let's the action know what transformations and libraries to test based on set of input events and the expected output, as well publish these transformations and libraries if the test passes.

      Meta file structure

     ```jsx
      // Meta file schema
      {
        "transformations" : <array of transformationSchema>,
        "libraries" : <array of librarySchema>
      }
     ```
      
     ```jsx
      // single transformationSchema
      {
        "file" (required): <path to the transformation code>,
        "name" (required): <transformation name>,
        "description" (optional): <transformation description>,
        "language" (required): <transformation language>,
        "test-input-file" (optional) : <path to file containing an array of events to test the transformation>,
        "expected-output" (optional) : <path to file containing an array of expected output for the above input after running the transformation code>
        "publish" (optional) : <set to true to automatically publish the transformation after testing (default: false)>,
      }
     ```
      
     ```jsx
      // single librarySchema
      {
        "file" (required): <path to the library code>,
        "name" (required): <library name: this is the name by which to import it in any transformation code>,
        "description" (optional): <library description>,
        "language" (required): <library language>,
        "publish" (optional) : <set to true to automatically publish the transformation after testing (default: false)>,
      }
     ```
      
     ```jsx
      // example meta.json
      {
        "transformations": [
          {
            "file": "./code/code.js",
            "name": "action-T1",
            "description": "javascript transformation T1",
            "language": "javascript",
            "test-input-file": "./code/events.json",
            "expected-output": "./code/expected.json",
            "publish": "true",
          },
          {
            "file": "./code/code_2.py",
            "name": "action-T2",
            "description": "python transformation T2",
            "language": "pythonfaas",
          }
        ],
        "libraries": [
          {
            "file": "./code/lib1.js",
            "name": "action-L1",
            "description": "javascript transformation library L1",
            "language": "javascript",
            "publish": "true",
          },
          {
            "file": "./code/lib2.py",
            "name": "action-L2",
            "description": "python transformation library L2"
            "language": "pythonfaas"
          }
        ]
      }
     ```

> Note:
-  Allowed values for language are `javascript` and `pythonfaas` (internal implementation to represent python)
-  All paths to files above should be relative to the base repo path