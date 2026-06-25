### Stop Rushing to Update Dependencies (Or: Don't Jump the Gun)

Every time a minor library patch is released, tools like Dependabot happily prompt us to hit *Merge*. But is this always a good idea?

**How Easy It Is to Break Everything:**

* **The Butterfly Effect (left-pad):** In 2016, a developer removed his `left-pad` package, just 11 lines of code, from npm. This broke thousands of project builds worldwide, including React and Babel. Vendoring completely protects against such scenarios.
* **Dependency Confusion Attacks:** If an attacker publishes a package to the public npm registry with the exact same name as your *private* internal package, but with a higher version number, the standard package manager will download the public (malicious) code by default. Apple, Microsoft, and PayPal all fell victim to this. In 2021, researcher Alex Birsan demonstrated how this works and collected over $130,000 in bug bounties.
* **AI in the Hands of Bad Actors:** The scheme is simple: first, one neat PR arrives, then another, the author asks to become a maintainer, and a few months later, a backdoor appears in the codebase. This is exactly how the `xz utils` attack worked. In the AI era, this scenario has become an order of magnitude cheaper. Almost anyone can now generate convincing commits, respond competently to code reviews, and run multiple accounts in parallel. For solo maintainers who are happy to receive any help, this is the most underestimated threat right now.
* **Google and the Monorepo:** Google stores almost all of its code in one giant monorepo. As a matter of principle, they vendor all external dependencies to guarantee that no external outage can halt their internal processes.

### The Illusion of Safe New Versions

When we update, we hope everything will run like clockwork. We run our tests, they're green, and it seems safe to deploy to production. But the problem is that changes within a package can be at a level where passing tests simply won't save you.

One of the main threats today is **Supply Chain Attacks**. Attackers are increasingly injecting malicious code directly into new versions of popular packages.

This means a fresh release can actually be much more dangerous than old code. Most malicious packages are pretty clumsy work: typosquatting (slightly altered names of popular packages).

Targeted attacks are a completely different story.
In 2018, malicious code in the **event-stream** package hung around on npm for about two months. In 2024, the backdoor in **xz utils** (a compression library found in almost every Linux distro) remained unnoticed for over a month, and the preparation took almost two years, the attacker had been systematically building a reputation as a legitimate maintainer since 2021.

## The Silent Threat: Install Scripts (postinstall)
One of the most dangerous mechanisms in the Node.js ecosystem is install scripts (preinstall, install, postinstall). They execute automatically the second a package lands in your dependency tree, no `import` or `require()` needed from your application. A simple typo, a hastily approved transitive dependency in a lockfile, or a compromised maintainer instantly becomes remote code execution.

Recent attacks have weaponized this exact vulnerability:
- September 2025 (Shai-Hulud worm): A self-replicating postinstall payload compromised 500+ npm packages by stealing maintainer tokens and republishing infected versions.
- September 2025 (chalk, debug, & 17 others): A phished maintainer account was used to inject Web3 wallet-draining code into packages with over 2 billion combined weekly downloads—all delivered via postinstall.
- March 2026 (Axios): A hijacked lead maintainer published versions with a "phantom dependency" that existed solely to trigger its postinstall hook and deploy a cross-platform Remote Access Trojan (RAT). The malicious package was never even imported into the Axios source code.

The End of an Era: npm Makes Scripts Opt-In

npm is the only remaining major package manager that runs dependency install scripts by default (pnpm v10+, Yarn Berry, Bun, and Deno already block them). But the security landscape has fundamentally shifted.

A new, [highly anticipated RFC](https://github.com/npm/rfcs/pull/868) is finally proposing to block these scripts by default during npm install. Projects will have to explicitly opt in via a new allowScripts field in package.json, and developers will manage this allowlist using new npm approve-scripts and npm deny-scripts commands. This supersedes older, rejected proposals (like 2021's [RFC #488](https://github.com/npm/rfcs/pull/488)) that were previously deemed "too disruptive." Today, the disruption of not doing this is far worse.

## **Just Don't Rush: The Power of a Simple Pause**
Just wait at least a few days (though this isn't always enough) before pulling in a fresh patch. This will filter out most primitive attacks. However, it won't save you from a truly targeted one.

### Vendoring: The Good Old Shield

How do you protect your project if you don't want to rely on the whims of external registries (npm, PyPI, etc.)? One solution is to **vendor your dependencies**.

This term refers to the practice of including the source code of external libraries directly into your repository, instead of relying on download links during the build process.

* **In JavaScript:** This can mean using Yarn Zero-Installs to store packages right in the repo, utilizing local archives via `npm pack`, patching (`pnpm patch`), or even creating your own Git forks of necessary libraries.

**Why do this?**

* **Absolute Autonomy:** If npm goes down or an author deletes their package, your CI/CD won't even notice.
* **Protection from Substitution:** You have 100% control over the code being executed.
* **Build Speed:** You don't waste time downloading thousands of tiny files from the internet.

Of course, there are downsides: the repository bloats, and you have to track updates manually (when they are truly needed).

---
### If You Update Directly from the npm Registry

**Lock your dependencies.**
Commit your lockfile (`package-lock.json`, `yarn.lock`, `pnpm-lock.yaml`) it pins the exact versions of your entire dependency tree. And make sure to use `npm ci` instead of `npm install` in your CI/CD pipelines:

* `npm install` can silently update the lockfile if there's a discrepancy with `package.json`, defeating the whole purpose of locking.
* `npm ci` installs exactly what is locked. If `package.json` and the lockfile are out of sync, it will fail with an error. This is a highly useful signal that something went wrong.
* **Bonus:** It runs faster because it doesn't spend time resolving dependencies.

**For Monitoring:** Use `npm audit` for basic CVEs, and tools like Socket.dev if you need behavioral analysis of a package, rather than just known vulnerabilities.
---
### Different Approaches & Updating Mindfully

Keeping an open-source library on the bleeding edge of the latest versions makes sense. But for Enterprise and SaaS products, chasing every minor patch is an unnecessary risk. Not every patch deserves your attention.

**Update consciously if:**

* You critically need a new feature.
* A patch is released that closes a known vulnerability (CVE) that *actually* impacts your code.
* You are planning a migration to a major version (you can't put this off for years, otherwise massive technical debt will build up and migration will become nearly impossible).
* A package is about to become deprecated.

For everything else, the good old engineering rule still applies: **"If it works (and isn't exposing security holes), don't touch it."**
