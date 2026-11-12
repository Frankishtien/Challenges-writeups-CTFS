# Agriweb


The vulnerability in the code you provided is **Prototype Pollution**. This is exactly the vulnerability type 
###  Vulnerability Analysis

The root cause lies in the `deepMerge` function:

```javascript

function deepMerge(target, source) {
  for (let key in source) {
    if (source[key] && typeof source[key] === 'object' && !Array.isArray(source[key])) {
      if (!target[key]) target[key] = {};
      deepMerge(target[key], source[key]);  // Recursive merge, no key filtering
    } else {
      target[key] = source[key];  // Direct assignment, no key filtering
    }
  }
  return target;
}

```

This function **does not filter dangerous keys**. When an attacker submits JSON data containing `__proto__`, `constructor`, or `prototype`, the `for...in` loop traverses these keys and ultimately pollutes `Object.prototype`.

In the AgriWeb challenge, the exploitation chain is as follows:

1\. A regular user's JWT **does not contain** the `isAdmin` attribute.

2\. The admin endpoint `/challenge/admin` checks `req.user.isAdmin`.

3\. The attacker sends a payload like `{"__proto__": {"isAdmin": true}}` to the profile update endpoint.

4\. `deepMerge` pollutes `Object.prototype.isAdmin = true`.

5\. Subsequently, all objects (including `req.user`) inherit `isAdmin: true`, thereby bypassing admin verification.

### Patch Solution

To pass the challenge, you need to modify `routes/profile.js` and add key filtering in `deepMerge`:

```javascript

const DANGEROUS_KEYS = ['__proto__', 'constructor', 'prototype'];

function deepMerge(target, source) {
  for (let key in source) {
    if (DANGEROUS_KEYS.includes(key)) continue;  // Skip dangerous keys
    if (source[key] && typeof source[key] === 'object' && !Array.isArray(source[key])) {
      if (!target[key]) target[key] = {};
      deepMerge(target[key], source[key]);
    } else {
      target[key] = source[key];
    }
  }
  return target;
}

```


---






