---
title: Github Permissions
navTitle: Github Permissions
---

# Github Permissions

There are a few levels of permissions that people and teams can have.

## Organization level

The [walmart-ios org][ios-org] has 335 folks in it and more every week. At the organization level there are only 2 permissions we care about, Member vs. Organizer. [Only 6 people are "owners" of the organization.][current-owners]

* Vasu Manepalli
* Sohil Rupani
* Joe Amrhein
* Eric Reedy
* Bharath Rao (IDC timezone support)
* John Regner (doing setup, adding new members)

(There are a few (9) bots that are also org owners, I'm reluctant to remove them because I do not understand the role that they play.)

You can read more about the [permissions granted to "Organization Owners" here][org-owner-docs]. The important permissions that we are likely to use are: 
* Invite people to join the organization
* Promote organization members to _team maintainer_
* Add and remove people from **all teams**
* Push (write) and clone (copy) _all repositories_ in the organization

[ios-org]: https://gecgithub01.walmart.com/walmart-ios
[current-owners]: https://gecgithub01.walmart.com/orgs/walmart-ios/people?query=role%3Aowner
[org-owner-docs]: https://docs.github.com/en/organizations/managing-peoples-access-to-your-organization-with-roles/roles-in-an-organization#permissions-for-organization-roles

## Team Level

From the team perspective there is much more flexibility. Each Team can have access to 1 or more repositories, and [the access they have is 1 of 5 levels. Read, Triage, Write, Maintain, Admin][team-access] It's worth pointing out at this point that any member of our org (even if they are not on a team) has "read" access to all repo's and can create new repositories.

[There](There) is a pretty [helpful chart here that shows the levels of access] for each of the 5 levels. We are mostly concerned with Write, Maintain, Admin.  Write is the default access that most teams have. This allows them to contributed by pushing their branch directly to the repository and performing PR's reviews and merging.

There are a few things that "Maintain" gives above write, with one being
critical:
 
* Push to protected branches

This same "access" is given to "Admin" teams too. So let's focus on that an not use the "Maintain" level.

Admin access gives all the possible actions including skipping branch protections and deleting the Repo.

The [full list of teams with access to the glass-repo are here][team-list].
There is one team with [admin access to the glass-ios repo: The `glass-platform-admin-ios` team][platform-admin] This team includes the Org owners listed above as well as:
 
* John Liedtke
* Drake Yundt

## Future Direction

There will be repositories added under the `glass-platform-admin-ios` team so that the team can help to manage them. More admin people might need to be added to the team (by an organization owner).

I'm happy to give up my Org Owner position, but then someone else would need to monitor `#glass-ios-new-developer-request` and add new developers to the org.

[team-access]: https://docs.github.com/en/organizations/managing-access-to-your-organizations-repositories/repository-roles-for-an-organization#repository-roles-for-organizationsh
[team-list]: https://gecgithub01.walmart.com/walmart-ios/glass-app/settings/access
[platform-admin]: https://gecgithub01.walmart.com/orgs/walmart-ios/teams/glass-platform-admin-ios/members
