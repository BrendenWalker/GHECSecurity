# Baseline GitHub Configuration

The following summarizes best practices and a proposed baseline security configuration for GitHub Enterprise Cloud using GitHub users (GitHub Enterprise Managed Users should be roughly similar).

Intended audience is Engineering Managers, Security Practitioners and developers who will be handling related permissions.

## Owner accounts for Enterprise and Organization

[GitHub enterprise best practices](https://docs.github.com/en/enterprise-cloud%40latest/admin/overview/best-practices-for-enterprises) and [GitHub best practices for organizations](https://docs.github.com/en/enterprise-cloud%40latest/organizations/collaborating-with-groups-in-organizations/best-practices-for-organizations) recommend assigning multiple owners to avoid risk associated with single point of failure.

In general I recommend assigned a trusted developer, one engineering manager and a trusted IT or security practioner.

### Organization Owners

Whether you need Organization level owners depends on need.  Start by reading [GitHub Strategies for using organizations in GitHub Enterprise Cloud](https://resources.github.com/learn/pathways/administration-governance/essentials/strategies-for-using-organizations-github-enterprise-cloud/).

Start with a single org and add additional if needed. A few examples that might make sense:

* An 'actions' org for shared workflows and actions.
* A 'workshop' for prototyping, validating any third party actions or any other work that shouldn't clutter the primary org.

## Enterprise Policies and Settings

[Enterprise policies](https://docs.github.com/en/enterprise-cloud%40latest/admin/policies/enforcing-policies-for-your-enterprise/about-enterprise-policies) allow us to enforce policy for all organizations.

### Policies
#### In Preview settings

There are some good things in preview currently. I recommend considering these on a limited/trial basis.
##### Repository

In preview feature that allows creation of more complex repository policies.  At the time this was written the feature was incommplete.  For example, the 'Add Actors' button indicates you can add roles, teams and apps however only Roles and Deploy Keys were listed.

Consider this feature in the context of [Repository Admin For Developers](#repository-admin-for-developers).

##### Code

Enterprise level branch/tag and push rulesets. You can target specific list, provide bypass, etc. 

###### Branch Rulesets

If feasible, I recommend targetting all organizations and all repositories.  No bypass.  Check Restrict deletions, Block force pushes and Require a pull request before merging.

###### Tag Rulesets

This is very situational.

###### Push Rulesets

Another area where it's going to depend on your particular needs.  

##### Code Insights

I'm not exacly sure what this is for.  It looks like you can create rules similar to the Code section above and this will provide a UI to review the rules.  What I find odd is that you can create rules here that are set to Enforce.  I'll have to dig into the documentation further.

##### Code ruleset bypasses

This is a new feature that appears to allow members to request bypass for a rule. I can imagine scenarios where this is very useful, however I don't believe this is relevant to baseline security configuration.

##### Custom Properties

No relevant to security baseline IMO.

#### Member privileges

* **Base Permissions**: Read. Every member has Read access to all repositories.
* **Repository creation**: Disabled. This may seem too restrictive, however allowing members to create repositories doesn’t seem to have much purpose in a corporate environment. If there is a need for a ‘skunkworks’, setting that up as a new organization provides better control.
* **Repository forking**: Disabled. Allowing members to fork repositories in a corporate environment can lead to a mess.
* **Outside collaborators**: Enterprise owners only, this is a high risk area.
* **Default branch name**: main but NOT enforced across the enterprise unless you are starting fresh.
* **Admin repository permissions** Controls what repository administrators are allowed to do. See [Repository Admin For Developers](#repository-admin-for-developers) for a deeper analysis.
  + **Repository visibility change**: Disabled. Blocks repository admins from changing repository visibility, for example to public.
  + **Repository deletion and transfer**: Disabled. I see no reason to allow admins to delete or transfer a repo.
  + **Repository issue deletion**: Disabled. I see no reason to delete issues, closing with some justification is preferred.

#### Codespaces

* **Manage organization access to GitHub Codespaces**: Disabled. “Only public repositories within this enterprise may use GitHub Codespaces”. Risk of code exposure is an issue, hence recommending Disabled.  Note that Disabled doesn't mean fully disabled, it's still allowed on public repositories.  

#### Copilot

* Disabled. Allowing Copilot would require deeper security analysis in context of specific policy.  

#### Actions

* **Policies**: Enable for all organizations
  + Allow enterprise, and select non-enterprise, actions and reusable workflows
* Allow actions created by GitHub. We may find that this is too restrictive.
* **Allow specified actions and reusable workflows**.  Some examples:
  + microsoft/\* - We’re allowing microsoft so we have access to official MSBuild and Azure related actions.
  + aws-actions/\* - Official Amazon WebServices actions. We may be able to drop this in the future.
  + azure/\* - Official Microsoft Azure actions.
  + docker/\* - Official docker actions (see SECC-17)
* **Runners** : Disable self-hosted runners for all organizations. This means that self-hosted runners cannot be managed at the repository level, requiring management at the Enterprise by owners. The risks of self-hosted runners are well documented in [About self-hosted runners]([https://docs.github.com/en/actions/hosting-your-own-runners/managing-self-hosted-runners/about-self-hosted-runners](https://docs.github.com/en/actions/hosting-your-own-runners/managing-self-hosted-runners/about-self-hosted-runners#self-hosted-runner-security).  
  + **Artifact and log retention**: 90 days (the default value). Organizations can set shorter duration, but not longer. This isn't really security sensitive, so feel free to change.
  + **Approval for running fork pull request workflows from contributors**: Require approval for all external contributors. If you've followed the rest of this guide it's not possible to fork anyway, however no reason not to set this tight as well.
  + **Fork pull request workflows in private repositories**: Run workflows from fork pull requests Unchecked (default). No sense allowing workflow runs in this scenario, another case were we're blocking with another policy.
  + **Workflow permissions**: Read repository contents and packages permissions. You may find that existing projects are comitting code during builds. If that's the case it should be reevaluate as it’s in general a bad practice
  + **Allow GitHub Actions to create and approve pull requests**: Unchecked. GitHub Advanced Security, DependaBot or Secrets Scanner may require this. Reevaluate if those features are required.

#### Hosted compute networking Policy

This is a relatively new feature: [About networking for hosted compute products in your organization](https://docs.github.com/en/organizations/managing-organization-settings/about-networking-for-hosted-compute-products-in-your-organization)

This feature has many potential security risks. Recommend disabling unless there is a specific need.

#### Projects

* **Organization projects**: No real security concerns unless you are enabling this on a public repository where bad actors could drop malware links or the like. 
* **Project visibility change permission**: No Policy needed.  This is already controlled with Enterprise Policy (if you follow this baseline).

#### Code security

* **Dependency Insights**: No Policy (default). This seems like a somewhat passive feature. If it’s annoying or cluttered UI then reconsider.
* **Enable or disable Dependabot alerts by repository admins**: Allowed (default). I see no reason to restrict this.
* **GitHub Advanced Security policies**
  * If you are using GHAS I recommend setting all options to Allowed and controlling via Billing settings with finer controls on Organization/Repository when needed.

#### Personal access token policies

This policy recently went live/productio and allows control over how PAT's are used. If you are in a high security situation controlling PAT's makes sense.  For baseline I would recommend leaving default values.

#### Sponsors

Entirely up to you.

### Settings

I'm only going to document security related items here. 

#### Authentication security

##### Two-factor authentication

Require checked. I would also recommend 'Only allow secure two-factor methods' IF possible.

#### SAML single sign on

Recommend using this if you can. Specific settings/configuration depend on your identify provider.

#### Code security

This alllows you to control various settings related to GitHub advanced security. I don't think it's sensible to consider this in the context of baseline security.

#### Audit Log

If possible you should configure log streaming to a secondary/secure service.  

#### Hooks

Webhooks are a potential security risk, mostly information disclosure. Enable only for specific need and follow best practices (https for example).

#### Hosted compute networking Settings

I'm not sure how this differs from the [enterprise policy](#hosted-compute-networking-policy).  In any case I don't think this is in scope for a generally usable security baseline.

#### Github Apps

You'll need to analyze risks for any apps you wish to use.  

## Organization Permissions

The organization [https://docs.github.com/en/organizations/managing-user-access-to-your-organizations-repositories/managing-repository-roles/setting-base-permissions-for-an-organization](https://docs.github.com/en/organizations/managing-user-access-to-your-organizations-repositories/managing-repository-roles/setting-base-permissions-for-an-organization#setting-base-permissions) setting will be configured with minimal required access and documented as necessary.

Access these from the organization Settings, Member privileges. Note that several settings are inaccessible due to being set by enterprise policy if you follow this baseline configuration.

The following recommendations from [OpenSSF best practices](https://best.openssf.org/SCM-BestPractices/) should be implemented:

* Actions
  + General
    - Policies: Set by enterprise policy.
    - Workflow Permissions
      * Read repository contents and package permissions.
  + **Allow GitHub Actions to create and approve pull requests**: UNCHECKED. No sense allowing workflows to bypass process.

## Repository Creation and Configuration

Your particular baseline configuration for repositories will depend on how your developers work best.  I'll offer the following as a sensible default.

### Creation

* For ‘Owner’ set an appropriate Organization. 
* **Visibility**: Private (note: Internal means visible to everyone in the Org, which just clutters the list)

### Settings

* **Default Branch**. Team choice, main preferred. NOTE: Only have options here if there is code present.
* **Features**
  + **Wikis** - Team choice
  + \*\* all other features disabled \*\*
* **Pull Requests** - NOTE: If you allow a team member Repo Admin rights they can adjust to accomodate needs. See  [Repository Admin For Developers](#repository-admin-for-developers) for more details.
  + **Allow merge commits** - CHECKED.
  + **Default commit message** set to Default message
  + **Allow squash merging** – UNCHECKED
  + **Allow rebase merging** – UNCHECKED
  + **Always suggest updating the pull request branches** – CHECKED
  + **Allow auto-merge** – CHECKED
  + **Automatically delete head branches** – CHECKED
* **Archives**
  + **Include Git LFS objects in archives**: CHECKED
* **Pushes**
  + **Limit how many branches and tags can be updated in a single push:** UNCHECKED

### Rules

Creating a default ruleset will make this trivial.  This is another area where team composition and workflow may require changes. As a good default I suggest the following:

Default main branch is protected with the following options CHECKED:

* **Restrict deletions** – Prevents deletion of main branch.
* **Require a pull request before merging** – Require a PR
  + **Require approvals** - 1 approval required.
  + **Require review from Code Owners** (only applies if files in commit have code owners)
  + **Require approval of the most recent reviewable push**
  + **Require conversation resolution before merging**
* **Block force pushes**

## Repository Admin For Developers

This can be a touchy area depending on policy and team composition.  Many of the risks of repository level Admin rights can be mitigated by Enterprise policy and/or Org settings.

