# Repository Admin Risks

The following table cover notable risks associated with Repository Admin permissions compared to Write permissions. 

In general I would recommend Admin permissions for a few developers to facilitate team workflows and encourage good CI/CD practices.

The first 4 columns cover potential impact of the risk, Confidentiality (C), Availability (A), Integrity (I) and Cost.

| C | A | I | Cost | Domain | Control/Accept | Permission | Risk | 
| --- | --- | --- | --- | --- | --- | --- | --- | 
| Y |  | Y |  | Access Control | Accept | Add a repository to a team | Potentially allow unauthorized access/modification. | 
| Y |  | Y |  | Access Control | Accept | Manage team and collaborator access to the repository | Potentially allow unauthorized access/modification.  Cannot add people outside of the organization, risk is allowing someone in house access to a repository that they shouldn't have access to. |
| Y |  |  |  | Code Security | Can be blocked by Enterprise Policy or Organization Settings | Manage the forking policy for a repository | Fork repo to private/public repo. Could expose data. |
| Y | Y |  |  | Code Security | Can be blocked by Enterprise Policy or Organization Settings | Delete or transfer repositories out of the organization | Deleting repo would impact availability, transfer could impact confidentiality |
| Y | Y | Y |  | Access Control | Can be blocked by Enterprise Policy or Organization Settings | Manage outside collaborator access to a repository | No limit of the potential impact here |
| Y |  | Y |  | Code Security | Can be blocked by Enterprise Policy or Organization Settings | Change a repository's visibility | Public could expose intellectual property. |
| ? |  |  | Y | Code Security | Can be blocked by Enterprise Policy or Organization Settings | Create codespaces for public/private repositories | Cost and possible confidentiality impact. |
| Y? |  |  |  | Client System Security | Can be blocked by Enterprise Policy or Organization Settings | Display a sponsor button | Possibly a bad actor could use this to direct to malware site, or exfiltrate limited metadata data such as Repo name. |
| Y | Y |  |  | Code Security | Can be blocked by Enterprise Policy or Organization Settings | Runners | Bad actor could add a runner that exfiltrates data or doesn't function properly resulting in loss of service. | 
| Y |  |  |  | Code Security | Can be blocked by Enterprise Policy or Organization Settings | GitHub Apps | External applications that are authorized to interact with a repository could exfiltrate data, possibly more. |
| Y |  |  |  | Code Security | Can be blocked by Enterprise Policy or Organization Settings | Action Permissoins | Allowing external actions from third parties could expose data. |
|  | Y |  | Y | Data Availability | Maximum set at the Org level. Repositories making this lower isn't an issue. | Artifact and Log Retension | Setting value too high could incur cost for additional services or prevent actions from running due to being out of storage space. | 
| Y |  |  |  | Code Security | Can be blocked by Enterprise Policy or Organization Settings | Actions Access | Possible code exposure. |
| Y |  | Y |  | Access Control | Can be partially Mitigated by Enterprise Policy | Manage individual, team, and outside collaborator access to the repository | Allowing outside collaborators could expose data as well as allow modifications |
|  | Y |  |  | Code Security | Accept | Make a repository a template | Minimal risk requiring ability to create repositories. |
|  | Y | Y |  | Code Security | Accept | Create, update, and delete GitHub Actions secrets | Replace secrets impacting availability, modify variables potentially impacting availability and integrity (loss of data). |
|  | Y |  |  | Code Security | Accept | Delete and restore packages | Deletion of package or restoring bad/incorrect package. Packages should all be built from source control anyway, allowing deletion/restore should not create additional risks. |
| Y | Y | Y |  | Protection Rules | Accept | Manage branch protection rules and repository rulesets | Impact would vary depending on use. If protecting a critical branch (such as a deployment branch) the impact could be severe.  Leaving these up to team and their workflow seems reasonable. Can be further mitigated with Repository Policy. |
|  | Y | Y |  | Protection Rules | Accept/Mitigate at Enterprise Level | Merge pull requests on protected branches, even if there are no approving reviews | Similar to 'Manage branch protection rules and repository rulesets'. |
|  | Y | Y |  | Protection Rules | Protection Rules - Feature Deprecated - Enterprise/Org | Delete tags that match a tag protection rule | Not a big issue in general. If you need to protect tags, it can be done at the Enterprise/Org level |
|  | Y |  |  | Platform Feature | Accept | Delete an issue | Normal team lead activity IMO.  |
|  |  | Y |  | Protection Rules | Accept | Edit the repository's default branch | Could change how branch protections function. Similar to protection rule management. |
|  | Y | Y |  | Protection Rules | Accept | Rename the repository's default branch | Could change how branch protections function. Similar to protection rule management. |
|  | Y |  |  | Platform Feature | Accept | Delete a discussion | Not security sensitive |
| Y |  |  |  | Code Security | Accept | Manage webhooks and deploy keys | Could be used to exfiltrate data. GitHub has rate limits so risk to availability should be minimal. |
|  |  | Y |  | Code Security | Mitigate at Enterprise/Org | Code Security Settings | Turning off could allow vulnerable software components into released software. This can be enforced at the Enterprise/Org level |
| Y |  |  |  | Code Security | Accept | Create autolink references to external resources | Potential to insert URL's into issues, pull request and commit data. Anyone with write access to the repository could do the same |
| Y | Y | Y |  | Deployment | Accept | Environments | Badly configured/deleted/modified environments could impact code deployabilit. User with write access could have similar impact. |
|  | Y |  |  | Code Security | Accept | Archive repositories | Makes repository read only impacting abilty to make changes. Self-foot-shoot. Recovery is to Unarchive |
