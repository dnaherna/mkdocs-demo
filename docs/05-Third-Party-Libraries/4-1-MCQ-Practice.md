---
title: Practice
---

## Multiple Choice

**Question 1:** Why do developers often rely on third-party libraries in software development?

- A. To avoid writing their own code entirely
- B. To reduce development time and reuse existing code
- C. To increase security and prevent vulnerabilities
- D. To maintain better control over project dependencies

Correct Answer: B

**Question 2:** What is the main risk associated with using outdated or abandoned third-party libraries?

- A. They consume excessive system resources
- B. They may become unavailable for installation
- C. They can contain unpatched vulnerabilities exploitable by attackers
- D. They require manual updates to integrate into projects

Correct Answer: C

**Question 3:** Which of the following measures helps minimize the risks associated with third-party libraries?

- A. Avoiding libraries from popular centralized repositories
- B. Using libraries without version specifications
- C. Ensuring all dependencies are installed in bulk
- D. Regularly checking for updates and security patches

Correct Answer: D

**Question 4:** When managing dependencies for a project, how could using the following version specification in package.json increase the risk of vulnerabilities?

```json
"dependencies": {
    "example-library": "*"
}
```

- A. It limits the project to only using outdated versions of the library.
- B. It introduces conflicts with other libraries due to incompatible versions.
- C. It allows the project to install any version, including potentially unpatched or vulnerable ones.
- D. It prevents the library from being installed successfully.

Correct Answer: C

**Question 5:** Consider the following snippet from a `requirements.txt` file for a Python project. What is the key issue with its dependency management strategy?

```text
example-library>=1.0
another-library==2.5.0
```

- A. Using `>=` for version specifications can lead to untested versions being installed.
- B. Specifying an exact version (e.g., `==2.5.0`) increases the risk of transitive dependency issues.
- C. Using `>=` prevents any updates, even critical security patches, from being applied.
- D. Specifying exact versions is not allowed in Python's requirements.txt files.

Correct Answer: A

**Question 6:** When analyzing a project's dependencies, you discover the following configuration. What hidden risk is MOST likely associated with the transitive dependencies (i.e., the dependencies of the project's direct dependencies) of this project?

```xml
<dependency>
    <groupId>com.example.lib</groupId>
    <artifactId>core-library</artifactId>
    <version>1.5.0</version>
</dependency>
```

- A. Transitive dependencies might have vulnerabilities unknown to the primary library.
- B. Transitive dependencies always introduce performance issues.
- C. Transitive dependencies do not receive updates unless manually specified.
- D. Transitive dependencies are automatically vetted by package managers for security.

Correct Answer: A

**Question 7:** Given the following code snippet, what is the main risk introduced by integrating a third-party library without vetting its source and update history?

```java
@PatchMapping("/update")
public void updateValue(@RequestParam("newValue") String newValue) {
    library.updateData(newValue);
}
```

- A. The library could introduce performance bottlenecks.
- B. The library could introduce hidden vulnerabilities, such as handling inputs insecurely.
- C. The library could fail to scale efficiently with increased usage.
- D. The library could prevent the code from compiling correctly.

Correct Answer: B

**Question 8:** What change occurred to `Polyfill.io` that enabled malicious activity?

- A. Developers failed to implement API rate limiting
- B. The polyfills contained deprecated features
- C. The original author used insecure hashing algorithms for polyfills
- D. The domain was sold to a company that compromised the service

Correct Answer: D

**Question 9:** How did Pdoc become vulnerable to attacks through `Polyfill.io`?

- A. It relied on `Polyfill.io` to provide backward compatibility for MathJax
- B. It included MathJax scripts from an insecure CDN
- C. It failed to validate user-generated documentation inputs
- D. It stored documentation in plaintext format

Correct Answer: A

**Question 10:** How did the attacker initially compromise `tj-actions/changed-files`?

- A. By using a compromised Personal Access Token (PAT) tied to a bot account.
- B. By exploiting a vulnerability in Coinbase's repository.
- C. By brute-forcing the administrator password for the repository.
- D. By injecting malicious JavaScript into the GitHub Actions workflow.

Correct Answer: A

**Question 11:** What was the purpose of the malicious run.sh script in the `tj-actions/changed-files` incident?

- A. To redirect users to phishing websites.
- B. To inject harmful code into production builds.
- C. To dump CI/CD secret credentials into GitHub Action log files.
- D. To enable unauthorized cloning of the repository.

Correct Answer: C
