# AirDC++ extensions

>This document defines the low-level AirDC++ extension specifications. If you are looking into developing an own extension, you might want to check out the [airdcpp-create-extension](https://github.com/airdcpp-web/airdcpp-create-extension) JavaScript starter project instead, which provides various abstractions for developers.

AirDC++ extensions use [AirDC++ Web API](http://apidocs.airdcpp.net) to communicate with the application via [WebSockets or HTTP REST calls](https://github.com/airdcpp-web/airdcpp-apidocs/blob/master/communication-protocols.md). Extensions are launched in a separate process with the specified [scripting engine](#scripting-engines) and their lifecycle and corresponding API sessions are managed by the application.

In case you have questions, you may use the issue tracker of this project or join the dev hub at adcs://web-dev.airdcpp.net:1511

## Resources

- [AirDC++ Web API reference](http://apidocs.airdcpp.net)
- [Example extensions for Javascript (WebSocket)](https://github.com/airdcpp-web/airdcpp-extension-js/tree/master/examples)
- [Example extension for Python 3.x (HTTP REST)](https://github.com/airdcpp-web/airdcpp-example-python-extension)
- [Extension starter project for JavaScript](https://github.com/airdcpp-web/airdcpp-create-extension)

## Public extension directory

The application uses [npm](https://www.npmjs.com) for public extension listing. The [extension structure](#extension-content) is also similar as the one used by npm packages.

## Scripting engines

Extensions are language-agnostic. However, [Node.js](https://nodejs.org) (JavaScript) is the preferred scripting engine as it will shipped with the future Windows application versions, and development utilities, public extension directory and most code examples are targeted for it.

Default scripting engines:

- *node* - [Node.js](https://nodejs.org)
- *python3* - [Python 3.x](https://www.python.org)

There must always be a corresponding engine configured with the same name in the application in order for the extension to be launched. The list of scripting engines is currently hardcoded but will be customizable in future application versions. 


## Startup parameters

AirDC++ will supply generic API and path information for launched extensions via command line arguments.

Example extension startup command:

```python3 C:\AirDC\Settings\extensions\airdcpp-example-ext\package\extension.py --name=airdcpp-example-ext --apiUrl=[::1]:5600/api/v1/ --authToken=b1f872e9-ddf8-4d55-98e4-ae1a024908cd --logPath=C:\AirDC\Settings\extensions\airdcpp-example-ext\logs\ --settingsPath=C:\AirDC\Settings\extensions\airdcpp-example-ext\settings\ --debug```


| Name | Type | Example value | Description
| :--- | :--- | :---: | :--- |
| **name** | `string` | airdcpp-example-ext | Name of the extension |
| **apiUrl** | `string` | [::1]:5600/api/v1/ | HTTP (non-TLS) URL that the application should use when accessing the API. The implementation should add the wanted protocol prefix (`ws://` or `http://`) based on the communication method being used. |
| **authToken** | `string` | b1f872e9-ddf8-4d55-98e4-ae1a024908cd | Session token to use for authentication. WebSockets should use the [socket authorization method](http://docs.airdcpp.apiary.io/#reference/sessions/authentication/socket) while the [`Authorization` HTTP header](https://github.com/airdcpp-web/airdcpp-apidocs/blob/master/communication-protocols.md#communicating-via-http) should be set with HTTP REST calls |
| **logPath** | `string` | C:\AirDC\Settings\extensions\airdcpp-example-ext\logs\ | Directory that can be used for saving extension-specific log files |
| **settingsPath** | `string` | C:\AirDC\Settings\extensions\airdcpp-example-ext\settings\ | Directory that can be used for saving extension-specific configuration files |
| **debug** | `boolean` | | If set, the extension may output additional information for debugging purposes |
| **signalReady** | `boolean` | | This flag will be added only if the [```airdcpp.signalReady```](#signalReady) package.json property is set to true. |


## Extension content


### Managing the extension structure with npm (recommended)

The [`npm` CLI utility](https://docs.npmjs.com/cli/npm) (usually shipped with [Node.js](https://nodejs.org)) may be used even when developing non-JavaScript extensions.

#### Initializing the project

If you want to create an extension from scratch, you may use the `npm init` command in an empty directory that will prompt you about the generic fields. 

#### Creating installable package

Executing `npm pack` will pack the extension with correct directory structure without publishing it to npm. 


### Content structure for manual packaging

```
package
│   package.json
```

All extension resources should be put inside a single directory (generally `package`) with `package.json` directly under it. For installation, the extension should be packed into a .tgz (.tar.gz) file.

Note that the name of the root package directory can be freely chosen. This enables direct installation of tagged releases from GitHub, assuming that the repository contains all files that are required for running the extension.



### package.json

Please see https://docs.npmjs.com/files/package.json for generic field descriptions.

Note that AirDC++ won't install possible external dependencies for extensions so all required resources should be shipped with the extension itself. 

#### Application-specific remarks

**Required fields**

###### `name`

Extension name. **The name must start with `airdcpp-`.**

##### `description`

Extension description

##### `version`

Extension version

##### `author`

Author's (user)name and possible email address

##### `main`

Script to execute by the application


**Optional fields**

##### `private`

If this field is set to `true`, the extension can't be accidentally published to `npm`. It also causes the application not to perform update checks for such extension from the npm registry. When writing extensions that you are not going publish on npm, it's important to enable this property so that your extension won't be replaced with another extension using the same name on npm due to updates offered to the user.

##### `keywords`

If you want the extension to be publicly listed in application's extension catalog, this field should contain the keyword `airdcpp-extensions-public`. If you don't want the extension to be listed publicly, you should omit this keyword.

##### `engines`

List of scripting engines to use for launching the extension. If the `engines` field is not set, the application will attempt to use the engine `node` by default.

Note that the application won't actually validate that the current engine version matches the wanted one. If needed, the extension should validate the engine version at runtime.

**airdcpp section (required)**

**Required fields**

##### `apiVersion` (*number*)

Target Web API version

**Optional fields**

##### `minApiFeatureLevel` (*number*, default value: ```0```)

Minimum Web API feature level supported by the extension.

##### `signalReady` (*boolean*, default value: ```false```)

Whether the extension will notify the application after it has completed initialization by calling the [/extensions/:id/ready](https://airdcpp.docs.apiary.io/#reference/extension-entities/methods/initialization-completed) API endpoint. By setting this property to ```true```, the extension can ensure that all the necessary hooks and listeners are in place before the application connects to the hubs or performs actions that involve running of hooks (such as validation non-shared bundles on startup or performing share refresh for roots for which the cached directory structure could not be loaded). It's recommended to send the ready signal right after the extension has performed the essetial operations for it to function correctly and perform possible long-running operations after that to avoid delaying the application startup. By default, the application will wait maximum of 5 seconds for the extensions to send the ready signal before continuing with the startup.

This field is supported with application versions with API feature level 6 or newer. When the value of the field is set to ```true``` and the property is supported by the current application version, the ```signalReady``` flag will be added in the extension startup parameters.
