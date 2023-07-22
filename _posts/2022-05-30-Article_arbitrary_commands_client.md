---
Author: 0xSoundOfSilence
Date: "2022-05-30"
Title: Command Injection found in game client
layout: post
---
# Summary 

XLab's reworked client IW4x is a modification of the fan-favourite Call of Duty game: Modern Warfare 2. Their mission is aimed towards fixing the flaws present in the original version of the game, which they open-source to involve the community as possible. So far, they've managed to build up a strong positive reputation among many players across multiple iterations of the CoD-franchise. I've personally tried some of their modified clients and I can tell you that they have done an incredible job with breathing new life into older CoD games.  

In this advisory we will be presenting a command injection PoC, which allows a threat actor to abuse certain features in-game that could run arbitrary code on the targeted host. This can be leveraged by the threat actor to in order to gain access to the system, attempt elevating its privileges in order to compromise the system.

The vulnerability exploits the Windows Shell File Association Vulnerability (CVE-2014-1807)

Hypothetical scenarios exploiting this vulnerability include:

- Setting up a network share containing a maliciously crafted executable and sending the path to that share through the discovered vulnerability.
- Accessing a link through the user's browser containing malicious code for staging an attack.
- Trigger a maintenance script that destroys / damages files on the system

The vulnerabilities and PoC's demonstrated in this advisory have been tested with IW4x versions 0.6.2 and 0.7.2. A patch has been issued by XLab and have mitigated this vulnerability with the release of version **0.7.3**. 

## Technical details:

### Code Analysis

The code that allows creates this vulnerability is part of a `Command.cpp` component under the `Modules` folder. In-game you're able to open a console for modifying properties of your game's instance. 

```cpp

Command::Command()
{
    AssertSize(Game::cmd_function_t, 24);

    Command::Add("openLink", [](Command::Params* params)
        {
            if (params->size() > 1)
            {
                Utils::OpenUrl(params->get(1));
            }
        });
}

```

The command calls the OpenUrl() function from its header file:

```cpp

void SafeShellExecute(HWND hwnd, LPCSTR lpOperation, LPCSTR lpFile, LPCSTR lpParameters, LPCSTR lpDirectory, INT nShowCmd)
{
    [=]()
    {
        __try
        {
            ShellExecuteA(hwnd, lpOperation, lpFile, lpParameters, lpDirectory, nShowCmd);
        }
        __finally
        {
        }
    }();
}
void OpenUrl(const std::string& url)
{
    SafeShellExecute(nullptr, "open", url.data(), nullptr, nullptr, SW_SHOWNORMAL);
}

```

## Proof-of-concept

Brief demonstration of the command launching a new process, which is then executed with the privileges of the game's process:

![https://imgur.com/8d5MtQw](https://imgur.com/8d5MtQw.png)


## Recommended Fix

To prevent command injection from occurring, commands should be sand-boxed to restrict access to resources and to prevent new processes from being started. Additionally, the in-game console should have the least amount of commands that interact directly with the OS, and limit its scope to only the in-game components.



## Final thoughts

It's interesting how video games can become such a severe risk for exploitation, given how immense popular video games have become over the years. Targeted attacks could jump the air gap to a private network, potentially risking more systems to get compromised. This vulnerability can be used to carry out sophisticated attacks, distribute malware or to launch a bot-net into the wild. 

---

references:
- [IW4x client](https://xlabs.dev/iw4x_download)
- [Diamante0018's github](https://github.com/diamante0018/)
- https://docs.microsoft.com/en-us/security-updates/securitybulletins/2014/ms14-027#severity-ratings-and-vulnerability-identifiers
- https://nvd.nist.gov/vuln/detail/CVE-2014-1807#vulnCurrentDescriptionTitle
	