# SolFuscator-node

[![Version](https://img.shields.io/npm/v/SolFuscator.svg)](https://www.npmjs.org/package/SolFuscator)
[![Downloads](https://img.shields.io/npm/dm/SolFuscator.svg)](https://www.npmjs.com/package/SolFuscator)
[![Bundle Size](https://img.shields.io/bundlephobia/minzip/SolFuscator)](https://bundlephobia.com/package/SolFuscator)
[![Build Status](https://github.com/SolFuscator/SolFuscator-node/actions/workflows/release.yml/badge.svg)](https://github.com/SolFuscator/SolFuscator-node/actions/workflows/release.yml)
[![License](https://img.shields.io/github/license/SolFuscator/SolFuscator-node)](LICENSE)

This repository hosts the official SDK for interacting with the SolFuscator API from Node.js environments.

**SolFuscator API access is only available for accounts under certain plans. For more information, please check out the pricing plans on the [SolFuscator website](https://lura.ph/#pricing).**

## Installation

Install the [SolFuscator](https://npmjs.org/package/SolFuscator) package from npm using your package manager of choice.

Example:
```sh
npm install SolFuscator
# or
yarn add SolFuscator
# or
pnpm install SolFuscator
```

## Usage

*The official [SolFuscator API documentation](https://lura.ph/dashboard/documents/apidoc) contains the most up-to-date and complete information and instructions for integrating with the SolFuscator API.*

#### [Basic](examples/basic.js)

```js
const { SolFuscator } = require("SolFuscator");

const apiKey = process.env.LPH_API_KEY; //replace with your api key
const SolFuscator = new SolFuscator(apiKey);

const obfuscate = async (script, fileName) => {
    console.log(`[*] file name: ${fileName}`);

    const nodes = await SolFuscator.getNodes();
    console.log(`[*] recommended node: ${nodes.recommendedId}`);

    const node = nodes.nodes[nodes.recommendedId];
    console.log(`[*] cpu usage: ${node.cpuUsage}`);

    const { jobId } = await SolFuscator.createNewJob(nodes.recommendedId, script, fileName, {});
    console.log(`[*] job id: ${jobId}`);

    const { success, error } = await SolFuscator.getJobStatus(jobId);
    console.log(`[*] job status: ${success ? "success" : "error"}`);

    if(success){
        const {fileName: resultName, data} = await SolFuscator.downloadResult(jobId);
        console.log(`[*] result name: ${resultName}`);
        
        return data;
    }else{
        throw error; //error is a string
    }
};

obfuscate("print'Hello World!'", `SolFuscator-node-${Date.now()}.lua`)
    .then(result => console.log(`[*] obfuscation successful: ${result.split("\n")[0]}`))
    .catch(error => console.error(`[*] obfuscation failed: ${error}`));
```

#### [List Options](examples/list_options.js)

```js
const { SolFuscator } = require("SolFuscator");

const apiKey = process.env.LPH_API_KEY; //replace with your api key
const SolFuscator = new SolFuscator(apiKey);

const listOptions = async () => {
    const nodes = await SolFuscator.getNodes();
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
const { SolFuscator } = require("SolFuscator");

const apiKey = process.env.LPH_API_KEY; //replace with your api key
const SolFuscator = new SolFuscator(apiKey);

const obfuscate = async (script, fileName) => {
    console.log(`[*] file name: ${fileName}`);

    const nodes = await SolFuscator.getNodes();
    console.log(`[*] recommended node: ${nodes.recommendedId}`);

    const node = nodes.nodes[nodes.recommendedId];
    console.log(`[*] cpu usage: ${node.cpuUsage}`);

    const { jobId } = await SolFuscator.createNewJob(nodes.recommendedId, script, fileName, {
        "TARGET_VERSION": "Lua 5.2"
    });
    console.log(`[*] job id: ${jobId}`);

    const { success, error } = await SolFuscator.getJobStatus(jobId);
    console.log(`[*] job status: ${success ? "success" : "error"}`);

    if(success){
        const {fileName: resultName, data} = await SolFuscator.downloadResult(jobId);
        console.log(`[*] result name: ${resultName}`);
        
        return data;
    }else{
        throw error; //error is a string
    }
};

obfuscate("print'Hello World!'", `SolFuscator-node-${Date.now()}.lua`)
    .then(result => console.log(`[*] obfuscation successful: ${result.split("\n")[0]}`))
    .catch(error => console.error(`[*] obfuscation failed: ${error}`));
```

## Documentation

*The official [SolFuscator API documentation](https://lura.ph/dashboard/documents/apidoc) contains the most up-to-date and complete information and instructions for integrating with the SolFuscator API.*

### class SolFuscator

#### constructor(apiKey: string): SolFuscator

Description: Creates an instance of the SDK.

Parameters:
- **apiKey** - API key to authenticate your requests. You can fetch your API key from your [account page](https://lura.ph/dashboard/account).

Returns: An instance of the SolFuscator class allowing API requests to be made.

---

#### getNodes(): Promise<{ nodes: {[nodeId: string]: SolFuscatorNode}; recommendedId: string | null }>

Description: Obtains a list of available obfuscation nodes.

Parameters: *None!*

Returns: *&lt;object&gt;*
- **recommendedId** - The most suitable node to perform an obfuscation based on current service load and other possible factors.
- **nodes** - A list of all available nodes to submit obfuscation jobs to. For more information on the structure of this field, please refer to the [SolFuscator API documentation](https://lura.ph/dashboard/documents/apidoc).

---

#### createNewJob(node: string, script: string, fileName: string, options: SolFuscatorOptionList, useTokens = false, enforceSettings = false): Promise<{ jobId: string }>

Description: Queues a new obfuscation task.

Parameters:
- **node**: The node to assign the obfuscation job to.
- **script**: The script to be obfuscated.
- **fileName**: A file name to associate with this script. The maximum file name is 255 characters.
- **options**: An object containing keys that represent the option identifiers, and values that represent the desired settings. Unless `enforceSettings` is set to false, all options supported by the node must have a value specified, else the endpoint will error.
- **useTokens** - A boolean on whether you'd like to use tokens regardless of your active subscription.
- **enforceSettings** - A boolean on whether you'd like the `options` field to require *every* option requested by the server to be present with a valid value. If this is false, your integration will not break when invalid options are provided; however, updates that change SolFuscator's options will result in your integration using default settings when invalid values are specified. By default, this is set to `true`.

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

### interface SolFuscatorOptionInfo

Fields:
- **name** - The human readable name associated with an option.
- **description** - The markdown formatted description for an option.
- **tier** - One of `CUSTOMER_ONLY`, `PREMIUM_ONLY`, `ADMIN_ONLY`.
- **type** - One of `CHECKBOX`, `DROPDOWN`, `TEXT`.
- **required** - If creating a user interface to integrate with SolFuscator, settings that contain a value of `true` for this field should be explicitly set by the user, since they have a high chance of causing incorrect output when not set properly.
- **choices** - An array of acceptable option values when `type == DROPDOWN`.
- **dependencies** - An array of required prerequisite values before this setting can be changed from the default value.

---

### interface SolFuscatorNode

Fields:
- **cpuUsage** - The current CPU usage of the node.
- **options** - An object with option identifiers as keys and `SolFuscatorOptionInfo` as values.

---

### interface SolFuscatorOptionList

An array of string keys, and values that may either be a string or boolean.

---

### interface SolFuscatorError

Fields:
- **param** - The parameter associated with the cause of the error.
- **message** - A human readable error message.

---

### class SolFuscatorException extends Error

Fields:
- **errors** - An array of `SolFuscatorError`.
- **message** - A human readable collection of error messages returned by a request.
