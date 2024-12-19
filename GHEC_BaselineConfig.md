# Baseline GitHub Configuration

The following summarizes best practices and a proposed baseline security configuration for GitHub Enterprise Cloud using GitHub users (GitHub Enterprise Managed Users should be roughly similar).

Intended audience is Engineering Managers, Security Practitioners and developers who will be handling related permissions.

## Owner accounts for Enterprise and Organization

[GitHub enterprise best practices](https://docs.github.com/en/enterprise-cloud%40latest/admin/overview/best-practices-for-enterprises) and [GitHub best practices for organizations](https://docs.github.com/en/enterprise-cloud%40latest/organizations/collaborating-with-groups-in-organizations/best-practices-for-organizations) recommend assigning multiple owners to avoid risk associated with single point of failure.

In general I recommend assigned a trusted developer, one engineering manager and a security pracitioner.

### Organization Owners

Whether you need Organization level owners depends on need.  Start by reading [GitHub Strategies for using organizations in GitHub Enterprise Cloud](https://resources.github.com/learn/pathways/administration-governance/essentials/strategies-for-using-organizations-github-enterprise-cloud/).  

## Enterprise Policies and Settings

[Enterprise policies](https://docs.github.com/en/enterprise-cloud%40latest/admin/policies/enforcing-policies-for-your-enterprise/about-enterprise-policies) allow us to enforce policy for all organizations.

Policies set at the Enterprise level are documented here, along with a brief justification or purpose.

### Policy Settings

#### Member privileges (BWalker – changed somewhere in 12/2025)

* **Base Permissions**: Read. Every member has Read access to all repositories.
* **Repository creation**: Disabled. This may seem too restrictive, however allowing members to create private repositories doesn’t seem to have a purpose in a corporate environment. If we find a need for a ‘skunkworks’, setting that up as a new organization provides better control.
* **Repository forking**: Disabled. Allowing users to fork repositories in a corporate environment can lead to a mess.
* **Outside collaborators**: Enterprise owners only. We need to keep control of this. Currently contractors have domain accounts, so there is no need for this feature.
* **Default branch name**: main but NOT enforced across the enterprise. We have many projects with different main branch names.
* **Admin repository permissions** These permissions control what repository administrators are allowed to do. At this time there are no plans to give this permission to anyone, however IF we find a need the following restrictions are in place.
  + **Repository visibility change**: Disabled. Blocks repository admins from changing repository visibility, for example to public.
  + **Repository deletion and transfer**: Disabled. I see no reason to allow admins to delete or transfer a repo.
  + **Repository issue deletion**: Disabled. I see no reason to delete issues, closing with some justification is preferred.

#### Codespaces

* **Manage organization access to GitHub Codespaces**: Disabled. “Only public repositories within this enterprise may use GitHub Codespaces”. NOTE: we are blocking the creation of repositories and changing visibility, there should not be any public repositories in our Enterprise

#### Copilot

* Disabled. Allowing Copilot would require security/vendor analysis. For now, disabled.

#### Actions

* **Policies**: Enable for all organizations
  + Allow enterprise, and select non-enterprise, actions and reusable workflows
* Allow actions created by GitHub. We may find that this is too restrictive.
* **Allow specified actions and reusable workflows**:
  + microsoft/\* - We’re allowing microsoft so we have access to official MSBuild and Azure related actions.
  + aws-actions/\* - Official Amazon WebServices actions. We may be able to drop this in the future.
  + bccactions/\* - Our internal organization that contains approved actions/shared workflows.. Etc.
  + azure/\* - Official Microsoft Azure actions.
  + docker/\* - Official docker actions (see SECC-17)
* **Runners** : Disable for all organizations. NOTE: this means that self-hosted runners cannot be managed at the repository level. Runners will need to be provisioned at the organization level.
* **Artifact and log retention**: 90 days (the default value). Organizations can set shorter duration, but not longer.
* **Fork pull request workflows from outside collaborators**: Require approval for all outside collaborators. We should not have any outside collaborators at this time.
* **Fork pull request workflows in private repositories**: Unchecked (default). No sense allowing workflow runs in this scenario, we already block creation of private repos for members.
* **Workflow permissions**: Read repository contents and packages permissions. We may find that existing projects are comitting code during builds. If so, that should be reevaluate as it’s in general a bad practice
* **Allow GitHub Actions to create and approve pull requests**: Unchecked. GitHub Advanced Security, DependaBot or Secrets Scanner may require this. Reevaluate when needed.

#### Projects

* **Organization projects**: Disabled. We use Jira for this purpose.
* **Project visibility change permission**: No Policy. Feature is not enabled.
* **Repository projects**: **Disabled**. Jira.

#### Code security and analysis

* **Dependency Insights**: No Policy (default). This seems like a somewhat passive feature. If it’s annoying or cluttered UI we can reconsider.
* **Enable or disable Dependabot alerts by repository admins**: Allowed (default). I see no reason to restrict this.

#### Personal access token policies (Beta)

* Leave default, to not Enroll currently. We may reevaluate after the feature is stable.

### Settings

#### Code security and analysis

* DependaBot
  + **Automatically enable for new repositories** - CHECKED. Can be disabled at the repo level.
* NOTE: Secret scanning is only available for public repositories. Private and Internal repositories will need to pay for Advanced Security to get similar function.

## Organization Permissions

The organization [https://docs.github.com/en/organizations/managing-user-access-to-your-organizations-repositories/managing-repository-roles/setting-base-permissions-for-an-organization](https://docs.github.com/en/organizations/managing-user-access-to-your-organizations-repositories/managing-repository-roles/setting-base-permissions-for-an-organization#setting-base-permissions)setting will be configured with minimal required access and documented as necessary.

Access these from the organization Settings, Member privileges. Note that several settings are inaccessible due to being set by enterprise policy.

The following recommendations from [OpenSSF best practices](https://best.openssf.org/SCM-BestPractices/) will be implemented:

* Actions
  + General
    - Policies: Set by enterprise policy.
    - Workflow Permissions
      * Read repository contents and package permissions.
  + **Allow GitHub Actions to create and approve pull requests**: UNCHECKED. No sense allowing workflows to bypass process.

## Repository Creation and Configuration

The following is meant to be a starting place. Teams will be provided leeway to adjust as required/approved.

### Creation

* For ‘Owner’ we currently have 3 organizations to choose from:
  + bccsoftware - all production software.
  + bccworkshop - testing, prototyping, etc. Limited access.
  + bccactions - actions and reusable workflows (third party or internally developed) that have been approved for use.
* **Visibility**: Private (note: Internal means visible to everyone in the Org, which just clutters the list)

### Settings

* **Default Branch**. Team choice, main preferred. NOTE: only appears if you have code checked in.
* **Features**
  + **Wikis** - Team choice
  + \*\* all other features disabled \*\*
* **Pull Requests** - NOTE: Teams can adjust this as required. If the team allows ‘Allow rebase merging’, they can also have a rule that requires linear history.
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

NOTE: importable rules are available here: https://github.com/bccsoftware/Infrastructure/tree/main/GitHub/Repository\_Rulesets

Default main branch is protected with the following options CHECKED:

* **Restrict deletions** – Prevents deletion of main branch.
* **Require a pull request before merging** – Require a PR
  + **Require approvals** - 1 approval required.
  + **Require review from Code Owners** (only applies if files in commit have code owners)
  + **Require approval of the most recent reviewable push**
  + **Require conversation resolution before merging**
* **Block force pushes**

# Custom Roles

To allow teams as much freedom as possible without giving Admin access, the following custom roles will be created:

**NOTE**: The assignment of these permissions will be on an as needed basis and documented as part of our normal Change Management process.

### Current Assignees

* Bernard Dijk
* Michael Ray
* Alex Trandaburu
* Clay Caragianes
* Tarr Soeung

## TeamLead

### Purpose

Allows select team members to configure repository rules.

### Risk(s)

Access to managing repository rules allows bypassing the rules.

### Benefit(s)

Allowing teams to define their own ‘guardrails’ can result in more effective rules while supporting team flexibility.

### Permissions

This role adds the following permissions to the Maintain role (note: the following is one permission in GitHub, details provided for clarity):

* Manage [branch protection rules](https://docs.github.com/en/enterprise-cloud%40latest/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches/managing-a-branch-protection-rule).
* Manage [tag protection rules](https://docs.github.com/en/enterprise-cloud%40latest/repositories/managing-your-repositorys-settings-and-features/managing-repository-settings/configuring-tag-protection-rules).
* Manage [repository rulesets](https://docs.github.com/en/enterprise-cloud%40latest/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/managing-rulesets-for-a-repository).
* Viewing [rule insights](https://docs.github.com/en/enterprise-cloud%40latest/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/managing-rulesets-for-a-repository).

### Current Assignees

* Bernard Dijk

## ManageActionVariables

### Purpose

Promote good CI/CD secrets and variables management.

Currently there is no way to set up a repository role with the ability to manage secrets and variables, so the only option at the repository level is Admin permissions which are simply too broad.

The organization level role allows the user to set organization action secrets and variables AND these can be restricted to use in specific repositories. This allows for sharing variables within the organization as necessary.

note: GitHub secrets are not viewable as plaintext. Once a secret is created, the only way to modify is to delete and re-create it.

### Risk(s)

A malicious actor with this permission could break builds or potentially redirect output if a variable is controlling build output location.

### Benefit(s)

Allowing select team members the ability to setup action variables and secrets is key to promoting good CI/CD security practices.

## Permissions

The ManageActionVariables role has the following permission:

* Manage organization wide actions [secrets](https://docs.github.com/en/actions/security-guides/using-secrets-in-github-actions) and [variables](https://docs.github.com/en/actions/learn-github-actions/variables).

### Current Assignees

* Bernard Dijk

# References

[Best practices for structuring organizations in your enterprise](https://docs.github.com/en/enterprise-cloud%40latest/admin/managing-accounts-and-repositories/managing-organizations-in-your-enterprise/best-practices-for-structuring-organizations-in-your-enterprise).

[GitHub best practices from Openssf.org](https://best.openssf.org/SCM-BestPractices/)

[More information on permissions for enterprise accounts](https://docs.github.com/en/enterprise-cloud%40latest/admin/overview/about-enterprise-accounts)


