# Solfuscator-node

[![Version](https://img.shields.io/npm/v/Solfuscator.svg)](https://www.npmjs.org/package/Solfuscator)
[![Downloads](https://img.shields.io/npm/dm/Solfuscator.svg)](https://www.npmjs.com/package/Solfuscator)
[![Bundle Size](https://img.shields.io/bundlephobia/minzip/Solfuscator)](https://bundlephobia.com/package/Solfuscator)
[![Build Status](https://github.com/Solfuscator/Solfuscator-node/actions/workflows/release.yml/badge.svg)](https://github.com/Solfuscator/Solfuscator-node/actions/workflows/release.yml)
[![License](https://img.shields.io/github/license/Solfuscator/Solfuscator-node)](LICENSE)

This repository hosts the official SDK for interacting with the Solfuscator API from Node.js environments.

**Solfuscator API access is only available for accounts under certain plans. For more information, please check out the pricing plans on the [Solfuscator website](https://sol.tor/#pricing).**

## Installation

Install the [Solfuscator](https://npmjs.org/package/Solfuscator) package from npm using your package manager of choice.

Example:
```sh
npm install Solfuscator
# or
yarn add Solfuscator
# or
pnpm install Solfuscator
```

## Usage

*The official [Solfuscator API documentation](https://sol.tor/dashboard/documents/apidoc) contains the most up-to-date and complete information and instructions for integrating with the Solfuscator API.*

#### [Basic](examples/basic.js)

```js
const { Solfuscator } = require("Solfuscator");

const apiKey = process.env.LPH_API_KEY; //replace with your api key
const Solfuscator = new Solfuscator(apiKey);

const obfuscate = async (script, fileName) => {
    console.log(`[*] file name: ${fileName}`);

    const nodes = await Solfuscator.getNodes();
    console.log(`[*] recommended node: ${nodes.recommendedId}`);

    const node = nodes.nodes[nodes.recommendedId];
    console.log(`[*] cpu usage: ${node.cpuUsage}`);

    const { jobId } = await Solfuscator.createNewJob(nodes.recommendedId, script, fileName, {});
    console.log(`[*] job id: ${jobId}`);

    const { success, error } = await Solfuscator.getJobStatus(jobId);
    console.log(`[*] job status: ${success ? "success" : "error"}`);

    if(success){
        const {fileName: resultName, data} = await Solfuscator.downloadResult(jobId);
        console.log(`[*] result name: ${resultName}`);
        
        return data;
    }else{
        throw error; //error is a string
    }
};

obfuscate("print'Hello World!'", `Solfuscator-node-${Date.now()}.lua`)
    .then(result => console.log(`[*] obfuscation successful: ${result.split("\n")[0]}`))
    .catch(error => console.error(`[*] obfuscation failed: ${error}`));
```

#### [List Options](examples/list_options.js)

```js
const { Solfuscator } = require("Solfuscator");

const apiKey = process.env.LPH_API_KEY; //replace with your api key
const Solfuscator = new Solfuscator(apiKey);

const listOptions = async () => {
    const nodes = await Solfuscator.getNodes();
    console.log(`[*] recommended node: ${nodes.recommendedId}`);

    const node = nodes.nodes[nodes.recommendedId];
    console.log(`[*] cpu usage: ${node.cpuUsage}`);
    
    console.log("[*] options:");
    for(const [optionId, optionInfo] of Object.entries(node.options)){
        console.log("  *", optionId, "-", optionInfo.name + ":");
        console.log("  |- desc:", optionInfo.description);
        console.log("  |- type:", optionInfo.type);
        console.log("  |- tier:", optionInfo.tier);
        console.log("  |- choices:", `[${optionInfo.choices.join(", ")}]`);
        console.log("  |- required:", optionInfo.required);
        if(optionInfo.dependencies) console.log("  |- dependencies: ", optionInfo.dependencies);
    }
};

listOptions();
```

#### [Setting Options](examples/set_options.js)

```js
const { Solfuscator } = require("Solfuscator");

const apiKey = process.env.LPH_API_KEY; //replace with your api key
const Solfuscator = new Solfuscator(apiKey);

const obfuscate = async (script, fileName) => {
    console.log(`[*] file name: ${fileName}`);

    const nodes = await Solfuscator.getNodes();
    console.log(`[*] recommended node: ${nodes.recommendedId}`);

    const node = nodes.nodes[nodes.recommendedId];
    console.log(`[*] cpu usage: ${node.cpuUsage}`);

    const { jobId } = await Solfuscator.createNewJob(nodes.recommendedId, script, fileName, {
        "TARGET_VERSION": "Lua 5.2"
    });
    console.log(`[*] job id: ${jobId}`);

    const { success, error } = await Solfuscator.getJobStatus(jobId);
    console.log(`[*] job status: ${success ? "success" : "error"}`);

    if(success){
        const {fileName: resultName, data} = await Solfuscator.downloadResult(jobId);
        console.log(`[*] result name: ${resultName}`);
        
        return data;
    }else{
        throw error; //error is a string
    }
};

obfuscate("print'Hello World!'", `Solfuscator-node-${Date.now()}.lua`)
    .then(result => console.log(`[*] obfuscation successful: ${result.split("\n")[0]}`))
    .catch(error => console.error(`[*] obfuscation failed: ${error}`));
```

## Documentation

*The official [Solfuscator API documentation](https://sol.tor/dashboard/documents/apidoc) contains the most up-to-date and complete information and instructions for integrating with the Solfuscator API.*

### class Solfuscator

#### constructor(apiKey: string): Solfuscator

Description: Creates an instance of the SDK.

Parameters:
- **apiKey** - API key to authenticate your requests. You can fetch your API key from your [account page](https://sol.tor/dashboard/account).

Returns: An instance of the Solfuscator class allowing API requests to be made.

---

#### getNodes(): Promise<{ nodes: {[nodeId: string]: SolfuscatorNode}; recommendedId: string | null }>

Description: Obtains a list of available obfuscation nodes.

Parameters: *None!*

Returns: *&lt;object&gt;*
- **recommendedId** - The most suitable node to perform an obfuscation based on current service load and other possible factors.
- **nodes** - A list of all available nodes to submit obfuscation jobs to. For more information on the structure of this field, please refer to the [Solfuscator API documentation](https://sol.tor/dashboard/documents/apidoc).

---

#### createNewJob(node: string, script: string, fileName: string, options: SolfuscatorOptionList, useTokens = false, enforceSettings = false): Promise<{ jobId: string }>

Description: Queues a new obfuscation task.

Parameters:
- **node**: The node to assign the obfuscation job to.
- **script**: The script to be obfuscated.
- **fileName**: A file name to associate with this script. The maximum file name is 255 characters.
- **options**: An object containing keys that represent the option identifiers, and values that represent the desired settings. Unless `enforceSettings` is set to false, all options supported by the node must have a value specified, else the endpoint will error.
- **useTokens** - A boolean on whether you'd like to use tokens regardless of your active subscription.
- **enforceSettings** - A boolean on whether you'd like the `options` field to require *every* option requested by the server to be present with a valid value. If this is false, your integration will not break when invalid options are provided; however, updates that change Solfuscator's options will result in your integration using default settings when invalid values are specified. By default, this is set to `true`.

Returns: *&lt;object&gt;*
- **jobId** - A unique identifier for the queued obfuscation job.

---

#### getJobStatus(jobId: string): Promise<({ success: true, error: null } | { success: false, error: string })>

Description: This endpoint does not return until the referenced obfuscation job is complete. The maximum timeout is 60 seconds, and this endpoint can be called a **maximum** of 3 times per job.

Parameters:
- **jobId** - The job ID of the obfuscation to wait for.

Returns: *&lt;object&gt;*
- **success** - A boolean indicating whether the job was successful.
- **error** - An error message if the job failed, or null if the job succeeded.

---

#### downloadResult(jobId: string): Promise<{ data: string; fileName: string }>

Description: Downloads the resulting file associated with an obfuscation job.

Parameters:
- **jobId** - The job ID of the obfuscation to download.

Returns: *&lt;object&gt;*
- **fileName** - A sanitized version of the initial filename, including a suffix to differentiate it from the original filename.
- **data** - The obfuscated script.

---

### interface SolfuscatorOptionInfo

Fields:
- **name** - The human readable name associated with an option.
- **description** - The markdown formatted description for an option.
- **tier** - One of `CUSTOMER_ONLY`, `PREMIUM_ONLY`, `ADMIN_ONLY`.
- **type** - One of `CHECKBOX`, `DROPDOWN`, `TEXT`.
- **required** - If creating a user interface to integrate with Solfuscator, settings that contain a value of `true` for this field should be explicitly set by the user, since they have a high chance of causing incorrect output when not set properly.
- **choices** - An array of acceptable option values when `type == DROPDOWN`.
- **dependencies** - An array of required prerequisite values before this setting can be changed from the default value.

---

### interface SolfuscatorNode

Fields:
- **cpuUsage** - The current CPU usage of the node.
- **options** - An object with option identifiers as keys and `SolfuscatorOptionInfo` as values.

---

### interface SolfuscatorOptionList

An array of string keys, and values that may either be a string or boolean.

---

### interface SolfuscatorError

Fields:
- **param** - The parameter associated with the cause of the error.
- **message** - A human readable error message.

---

### class SolfuscatorException extends Error

Fields:
- **errors** - An array of `SolfuscatorError`.
- **message** - A human readable collection of error messages returned by a request.

## Useful Links
- [Visit the Solfuscator Website](https://sol.tor/ "Solfuscator - Online Lua Obfuscation")
- [Join the Solfuscator Discord](https://discord.sol.tor/ "Solfuscator Discord Server")
- [Read the Solfuscator Documentation](https://sol.tor/dashboard/documents "Solfuscator Documentation")
- [Read the Solfuscator FAQs](https://sol.tor/dashboard/faq "Solfuscator Frequently Asked Questions")
