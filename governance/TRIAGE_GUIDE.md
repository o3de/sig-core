# O3DE SIG-Core - Issue Triage Guide

## Overview
This guide covers how to triage GitHub issues for SIG-Core.

Maintainers are encouraged to reference and update this guide to ensure
all contributors to SIG-Core understand how issues are handled and accepted by the SIG.

This guide uses the [general SIG triage guide](https://github.com/o3de/community/blob/main/sigs/docs/sig-issues-triage-guide.md) as a base.

##  Issue triage
Triaging is the process used to handle intake of issues into the SIG-Core backlog. The process aims to ensure issues are both relevant to SIG-Core
and contain sufficient information so that the community can take action.

Process aims to ensure that:
* Issues are appropriate for SIG-Core. Confirms that issues are actual issues, rather than requests for help or issues for another SIG.
  * The [SIG-Core Charter](https://github.com/o3de/sig-core/blob/main/governance/SIG%20Core%20Charter.md) can be used as a basis to determine relevant issues for the group
* Issues have clear information to enable SIG-Core to address the problem or request.
* Issues are regularly maintained and updated until they are resolved.  
* Issue load is balanced across SIG maintainers when action is required.
* All the SIG-Core community can participate.

# Triage process
SIG-Core triages issues once a week on [Wednesdays at 9:00AM PST (November-March) / 9:00AM PDT (March-November)](https://lists.o3de.org/g/o3de-calendar/viewevent?repeatid=39865&eventid=1289822&calstart=2021-12-29). Anyone is welcome to attend. Triage will be led by SIG chair or co-chair (referred to below as *Triage Leader*)


## Links for triage
1. Open issues with `needs-sig` label in o3de/o3de repo: https://github.com/o3de/o3de/issues?q=is%3Aissue+is%3Aopen+label%3Aneeds-sig+sort%3Acreated-asc
1. Open issues with `needs-sig` label in o3de/o3de-extras repo: https://github.com/o3de/o3de-extras/issues?q=is%3Aissue+is%3Aopen+label%3Aneeds-sig+sort%3Acreated-asc
1. Main O3DE repo: https://github.com/o3de/o3de/issues?q=is%3Aissue+is%3Aopen+label%3Aneeds-triage+label%3Asig%2Fcore+sort%3Acreated-asc
1. O3DE Extras repo: https://github.com/o3de/o3de-extras/issues?q=is%3Aopen+label%3Aneeds-triage+label%3Asig%2Fcore+sort%3Acreated-asc

## Triage leader guide
1. Join the SIG-Core discord [voice channel](https://discord.com/channels/805939474655346758/849888184045010944)
1. Announce yourself as the Triage Leader and wait a few minutes for others to join the call.
1. Use the *Individual Issue Triage* guide below to process all new issues for SIG:
   1. Review any open issues with the "needs-sig" label that may be for SIG-Core in the [o3de/o3de](https://github.com/o3de/o3de/issues?q=is%3Aissue+is%3Aopen+label%3Aneeds-sig+sort%3Acreated-asc) repo
       1. Remove `needs-sig` and add `sig/core`. These items will now show up when reviewing issues below.
   1. Review any open issues with the "needs-sig" label that may be for SIG-Core in the [o3de/o3de-extras](https://github.com/o3de/o3de-extras/issues?q=is%3Aissue+is%3Aopen+label%3Aneeds-sig+sort%3Acreated-asc) repo
   1. Process all [o3de/o3de](https://github.com/o3de/o3de/issues?q=is%3Aissue+is%3Aopen+label%3Aneeds-triage+label%3Asig%2Fcore+sort%3Acreated-asc) repo issues for SIG-Core
   1. Process all [o3de/o3de-extras](https://github.com/o3de/o3de-extras/issues?q=is%3Aissue+is%3Aopen+label%3Aneeds-triage+label%3Asig%2Fcore+sort%3Acreated-asc) repo issues for SIG-Core

If there are questions about what to do with an issue please raise questions with SIG Chair(s) or start a conversation in SIG-Core.  
Please triage issues from oldest to latest to ensure older issues are not ignored for multiple triage sessions.

### Individual issue triage guide

#### General triage guidance
* (Optional but highly recommended) Share you screen so others can follow along.
* Announce issue number and title to those in Discord voice channel, so those who are just listening can also follow along
* Pause to let others read and process the information.
* Always Add comments to issue, when appropriate, to capture issue triage decisions.
* Ping other SIG-\* groups when appriopriate in comments when needed

#### Issue triage 
1. Ensure issue is for SIG-Core ([o3de/o3de](https://github.com/o3de/o3de/issues) and [o3de/o3de-extras](https://github.com/o3de/o3de-extras/issues))
   1. If the issue is not for SIG-Core, remove the `sig/core` label and comment on the issue as to reason why issue is not for SIG-Core. 
   1. If the correct SIG is known, add that SIG label to the issue. Otherwise, add the `needs-sig` label so the general O3DE issue triage meeting can find the appropriate owners. 
1. Review the issue and any comments to see if it can be accepted by SIG.
   1. **If issue is a bug**: Check that report has enough information for someone to reproduce or understand the issue.
   1. **If issue is a feature request**: Review the technical implications of the request. 
      1. If it's a large change then issue should become an RFC or be brought to SIG-Core meeting for discussion. Ask requestor to bring issue back as RFC or start a discussion topic. Add the issue to the next SIG-Core meeting agenda, if that would be more appropriate. 
1. If issue can be **accepted** then:
   1. If issue is a bug, add the `kind/bug` label.
   1. If issue is for a feature request, add either `kind/feature` or `kind/enhancement`. Add `feature/*` labels as appropriate.
   1. Set a priority for issue based on impact (ask other SIG members on call for guidance).  
   1. Mark the issue as `triage/accepted`.
1. If the issue **requires more information** or is **rejected**, then:
   1. Assign a reviewer, if required, to handle follow-up comments, to reproduce the issue or ask for further clarifying information.
   1. **If issue is rejected**: Reviewer/triage leader should reject issue and provide reason for rejection. 
      1. Mark the issue as `triage/declined`.
   1. **If issue needs more information**: Reviewer/triage leader should add clear comments requesting the additional information. 
      1. Mark the issue with `triage/needs-information`. Its recommended that all issues in this state have an assigned reviewer who will track updates until all required information is received. Issue can then be reconsidered for acceptance.
1. Remove the `needs-triage` label from issue if SIG/Core is only assigned SIG.
   1. See [general guidance](https://github.com/o3de/community/tree/main/sigs/docs) for issues requiring input from multiple SIGs.

### Additional Labels to consider for contributors
* Consider adding the [`good first issue`](https://github.com/o3de/o3de/labels/good%20first%20issue) label to identify issues that have straightforward/simple fixes for new contributors to fix. Examples could include config, docs, comments and testing changes.
* Consider adding the [`help-wanted`](https://github.com/o3de/o3de/labels/help-wanted) label for issues that do not have immediate resourcing and contributions by others would be welcome.

## Additional triage tasks
If time permits select one or more of the following tasks:

* Review all open [bugs without acceptance](https://github.com/o3de/o3de/issues?q=is%3Aissue+is%3Aopen+label%3Asig%2Fcore+-label%3Atriage%2Faccepted) and ensure they have `triage/accepted`.
  * Ensures issue that have been assigned to SIG have been triaged correctly. 
* Review any open [blocker and critical](https://github.com/o3de/o3de/issues?q=is%3Aissue+is%3Aopen+label%3Akind%2Fbug+label%3Apriority%2Fcritical%2Cpriority%2Fblocker+label%3Asig%2Fcore) issues in the o3de/o3de repository:
  * Ensures priority is still valid. 
  * Ensures issues are still valid.
  * Assign any required commentators or ask for updates.
* Review any PRs for the SIG that are more than [30 days old](https://github.com/o3de/o3de/pulls?q=is%3Apr+is%3Aopen+label%3Asig%2Fcore+sort%3Acreated-asc).
  * Ensures PRs are progressing and are not blocked on contributor or maintainer action.
* Review issues open for more than [90 days old](https://github.com/o3de/o3de/issues?q=is%3Aissue+is%3Aopen+label%3Asig%2Fcore+sort%3Acreated-asc).
  * Ensures issues are still relevant to SIG. 


## Issue Workflow
If you are assigned an issue to validate or reproduce, work with requestor to get enough information to validate the issue.

If it can be reproduced then:
* Add comment to confirm reproduction.
    * Can work with SIG-Chair(s) to add `triage/accepted` labels and define priority or wait for next issue triage meeting.
* Ensure issue is not a duplicate.

If issue cannot be reproduced then:
* Comment on the issue and ask the requester for more information to aid reproduction, add the `triage/needs-information` label.
* Or close the issue if both parties agree that this is no longer an issue or not reproducible.

If the issue is not clear or needs more information:
* Comment on the issue and add the `triage/needs-information` label to show that the requestor needs to provide more information.

# Stale or abandoned issues
SIG will periodically audit for stale items. If during triage, you encounter stale issues, use the guidance below to see if issue should be closed.

## SIG assigned but no action
If an issue with the SIG-Core label has had no updates for a while (14 days), follow-up with the SIG, either through Discord chat channel, triage or standard meeting. Consider attending a SIG-Core meeting to raise the issue for discussion.

## No activity for 90 days
An issue can be removed if it has been abandoned by the requestor. Issues are considered abandoned if there has been no activity for *90* days, especially if issue has had `triage/needs-information` label applied and there has been no follow-up from issue reporter.

## FAQ
1. What should I do if triage rejects my issue?
   1. Issues should be rejected with clear comments that provide reason for rejection. If you disagree or want to discuss the reason please start a chat in SIG-Core or add as an agenda item for SIG-Core's public meeting.
   2. If you still do not support SIG-Core's decision, then please raise with the [O3DE TSC](https://github.com/o3de/tsc).
2. What should I do if I have an urgent issue that cannot not wait for public issue triage?
   1. Please raise the issue in SIG-Core chat and ask for triage, this is so all SIG has visibility.
   2. SIG-Chair(s) can appoint a reviewer to ensure issue is triaged as soon as possible.
   3. If the intent is for you to work on the issue immediately, then please self-assign or work with a maintainer to assign.

## Acknowledgments
This SIG-Core guide was inspired by the SIG-Network triage guide located at https://github.com/o3de/sig-network/blob/main/TRIAGE_GUIDE.md

