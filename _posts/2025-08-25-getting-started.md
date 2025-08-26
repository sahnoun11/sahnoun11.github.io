---
layout: post
title: CVE-2022–45875 Command Injection Vulnerability in Apache DolphinScheduler
categories:
- Red Teaming
tags:
- Red Team
- Pentest
- HTB
- CVE
- Bug Bounty
date: 2025-08-25 01:00 +0100
description: In this article, we dive deep into a critical security flaw in Apache DolphinScheduler (CVE-2022-45875) a command injection vulnerability within its alert script execution module. This weakness allows attackers to inject and execute arbitrary commands, potentially leading to remote code execution (RCE) and full system compromise.
---

## Introduction

Apache DolphinScheduler is a widely used distributed and visual workflow scheduler designed to orchestrate complex data pipelines. However, a severe command injection vulnerability (CVE-2022–45875) was discovered in its alert script execution module. This security flaw can allow an attacker to execute arbitrary commands, potentially leading to remote code execution (RCE).

This article delves into the technical details of the vulnerability, how it is exploited, and how to mitigate the risk. We will also answer specific questions related to the affected code.

### Affected Component

The vulnerability resides in the ScriptSender class of the dolphinscheduler-alert-script module. The affected code is present in the following file: [View vulnerable code on GitHub](https://github.com/apache/dolphinscheduler/blob/3.0.0-release/dolphinscheduler-alert/dolphinscheduler-alert-plugins/dolphinscheduler-alert-script/src/main/java/org/apache/dolphinscheduler/plugin/alert/script/ScriptSender.java)

#### Vulnerable Code Snippet :
```html
private AlertResult executeShellScript(String title, String content) {
    AlertResult alertResult = new AlertResult();
    alertResult.setStatus("false");
    if (Boolean.TRUE.equals(OSUtils.isWindows())) {
        alertResult.setMessage("shell script not support windows os");
        return alertResult;
    }
    
    File shellScriptFile = new File(scriptPath);
    if (!shellScriptFile.exists()) {
        logger.error("shell script not exist : {}", scriptPath);
        alertResult.setMessage("shell script not exist : " + scriptPath);
        return alertResult;
    }
    
    String[] cmd = {"/bin/sh", "-c", scriptPath + ALERT_TITLE_OPTION + "'" + title + "'" + ALERT_CONTENT_OPTION + "'" + content + "'" + ALERT_USER_PARAMS_OPTION + "'" + userParams + "'"};
    int exitCode = ProcessUtils.executeScript(cmd);
    
    if (exitCode == 0) {
        alertResult.setStatus("true");
        alertResult.setMessage("send script alert msg success");
        return alertResult;
    }
    alertResult.setMessage("send script alert msg error,exitCode is " + exitCode);
    logger.info("send script alert msg error,exitCode is {}", exitCode);
    return alertResult;
}
```

#### How the Vulnerability Works

The command injection flaw occurs because user-controlled inputs (```title```, ```content```, and ```userParams```) are concatenated into a shell command string and executed via ```ProcessUtils.executeScript(cmd)```.

Since these variables are not sanitized or properly escaped, an attacker could inject arbitrary shell commands into any of these parameters. When the cmd array is executed, these injected commands will run with the same privileges as the application, potentially leading to a full system compromise.

#### Exploitation
An attacker can exploit this vulnerability by crafting malicious inputs that execute arbitrary shell commands. For example, passing the following as the ```title``` parameter:

```bash
'; rm -rf / -- '
```
![Sukuna](/assets/img/1*V8Xp67WUnK6FxFq33QCzXg.jpeg)

Would result in the following command execution:

``` bash
/bin/sh -c /path/to/script.sh -t ''; rm -rf / -- '' -c 'content' -p 'userParams'
```
This could lead to catastrophic consequences, such as deleting critical system files or opening a reverse shell to gain unauthorized access.

#### Answering Key Questions
* What is the name of the variable that stores the array of commands?
 The variable storing the command array is ```cmd```.

* What method is called to execute the commands in the array?
 The method responsible for executing the command array is ```ProcessUtils.executeScript```.

* What are the inputs included in constructing the commands?
 The inputs used in constructing the commands are ```scriptPath```,```title```,```content```, ```userParams```.

#### Mitigation
To mitigate this vulnerability, Apache DolphinScheduler should implement proper input sanitization and validation. Here are some key mitigation strategies:

#### Parameter Escaping and Validation
Instead of directly concatenating user input into the command string, properly escape or validate inputs to prevent command injection. Using a predefined list of allowed characters or employing safe shell argument handling mechanisms can prevent exploitation.

#### Using a Secure API
Instead of executing shell commands directly, use secure alternatives such as Java’s ProcessBuilder with separate arguments to avoid string concatenation vulnerabilities.

```html 
ProcessBuilder pb = new ProcessBuilder("/bin/sh", "-c", scriptPath, "-t", title, "-c", content, "-p", userParams);
Process p = pb.start();
```
#### Least Privilege Principle
Upgrade to the latest patched version of Apache DolphinScheduler. The vulnerability has been addressed in newer releases.

### Conclusion
CVE-2022–45875 is a critical command injection vulnerability in Apache DolphinScheduler that could allow an attacker to execute arbitrary commands. By understanding how the vulnerability arises and implementing the suggested mitigation strategies, organizations can protect themselves from potential exploits.

Security teams should prioritize patching, input validation, and secure execution practices to ensure the safety of their systems. Always remember: Never trust user input without proper sanitization!

Stay safe and secure! 🚀

![Sukuna](/assets/img/1*iw913rFgrhRp5wor5WgADw.jpeg)