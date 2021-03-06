---
layout: default
---
{% include links %}
* TOC
{:toc}

# Support scope #
SeLite doesn't support functionality of related components and technologies (Selenium IDE, SelBlocks, SQLite, Firefox, Selenium selectors, XPath, Javascript...). See also [AboutDocumentation](AboutDocumentation) > [Documentation scope](AboutDocumentation#documentation-scope).

Search in current [issues](https://github.com/selite/selite/issues). If an existing issue covers your problem, vote for it. See also [ThirdPartyIssues](ThirdPartyIssues) > [Where to vote](ThirdPartyIssues#where-to-vote).

# Report an issue #
First, follow [TroubleShooting](TroubleShooting). Before you [report an issue](https://github.com/SeLite/SeLite/issues/new), note details and versions of

  * any SeLite extensions
  * Selenium IDE
  * any other Firefox extensions
  * Firefox (and if it's not a current release then also URL from where you downloaded it)
  * SQLite DB (if applicable)
  * OS and whether 32 or 64 bit
  * any non-SeLite components and their origin - the URL for download or GIT/SVN checkout (including branch and revision)

Collect any errors from logs (as per [TroubleShooting](TroubleShooting)). If logs are short, copy and paste them in the error report. Otherwise install [AddOnsThirdParty](AddOnsThirdParty) > File Logging, restart Firefox, re-run your script, save the log in a file and attach it to the error issue.

By submitting an issue or a comment you agree that project administrator(s) can, without notice, edit your comment, and/or incorporate your comment into SeLite, under the current or any future license.

<!-- too much:
* apply [https://developer.mozilla.org/en-US/docs/Mozilla/QA/Bug_writing_guidelines Bug Writing Guidelines] from Mozilla
* follow [https://bugzilla.mozilla.org/page.cgi?id=etiquette.html Etiquette] of Bugzilla
* create reports as [http://sscce.org/ Short, Self Contained, Correct Examples]
-->

## Submit any failing scripts
Save your [scripts][script] ([cases][case] and [suites][suite]), along with any HTML forms/pages as per [PackagedScripts](PackagedScripts) > [Structure](PackagedScripts#structure). If needed, include any [Settings](Settings) as per {{navValuesManifests}}. Put that in a `.zip` file and attach it to the issue. That prevents typing mistakes and misunderstanding. Alternatively, if you report only a few simple Selenese command calls, you could send them as text, e.g.:

```
command | target | value
command | target | value
```

However, that allows for mistakes on both sides. Also, the system for tracking issues may modify text content (it formats it as Markdown, it auto-links #123.. to issue numbers and it auto-links URLs).

Do not just attach screenshots that just show messages or your command(s). Also copy and paste those messages as text.

## Submit your Firefox profile ##
If the problem only happens with third party or custom extensions, or if it involves a complex situation, you may need to attach your Firefox profile. See also [TroubleShooting](TroubleShooting) > [Separate Firefox profile](TroubleShooting#separate-firefox-profile). [Locate the Firefox profile](https://support.mozilla.org/en-US/kb/profiles-where-firefox-stores-user-data#w_how-do-i-find-my-profile) folder. Turn Firefox off, remove any [private data](https://support.mozilla.org/en-US/kb/recovering-important-data-from-an-old-profile#w_your-important-data-and-their-files) and `places.sqlite`.

However, GitHub doesn't allow issue attachments other than images. So, zip up the folder, upload it to somewhere on the internet and add a link to it to the issue. Alternatively, and only if the `.zip` file is under 10MB, email it to the maintainer.

# GitHub references
Please, refer to other issues, projects, forks, commits and pull requests with GitHub [Autolinked references and URLs](https://help.github.com/articles/autolinked-references-and-urls/).