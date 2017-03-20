# 1) Unreviewed Design Notes/Thoughts

__As an example, the following notes are assuming a 2018 US national election.__

See [project-overview.md](https://github.com/PacemTerra/votes/docs/project-overview.md) for a general overview.

There is a local copy of [NIST.SP.1500-100.pdf](https://github.com/PacemTerra/votes/docs/NIST.SP.1500-100.pdf) which contains a terminology section on page 120 as well as the precinct and ward map for Cambridge Massachusetts on pages 14 and 15.

## 1.1) Basic Git Repo Prototype

As a prototype implementation the VOTES framework can (in theory - TBD) be implemented as a set of git repos connected via submodules.  The parent repo is associated with the __Geopolitical Geographical Overlay (GGO)__ with the greatest geographical geopolitical extent in the election.  For the 2018 US national election, this would be the federal government as it is a national election.

Note that if only one or a few states adopt VOTES by that time, it is ok. Those states would act as the VOTES root certificate authorities for the towns and counties/boroughs that are participating.

All VOTES repos have the same layout regardless of the GGO (national, state, county, town, etc).  What is different is only the data that is contained in the folder structure within and where/how the submodule tree is stitched together.

Each repo contains the information authored by each GGO.  However, per the git subtree hierarchy children and parent repos as well as configuration data information is shared across repo.

# 2) Folder Layout Per Git Repo

So the root most GGO is that git repo _owned/authored_ by the VOTES root authority of the cert chain for the 2018 US election (_the election_).  In this case the agency deemed responsible for actually running the 2018 national US election would be the VOTES root authority.  The folder structure for VOTES repos is:

#### ./info

Contains configuration data for this GGO in yaml form (unless it really needs to be xml or something else).  Yaml is more human readable and more (git) merge-able.  If so necessary it can be translated into xml when needed.

#### ./bin

Contains executables directly associated with the VOTES repos are located in this folder.  Includes the executables necessary to tally the various contests and leverages configuration data.  For example the tally scheme for a GGO or a contest within the GGO can be configured in ./info while the actual algorithms and tally code is implemented in ./bin.

For a US presidential election, due to the electorial college the states tally function will mirror/implement the states electorial tally function, which is basically a summation.  However different states do it differently, and in the case of the electoral college the electors actually must vote, so the tally function non binding and perfunctory only.

This folder does not contain phone or desktop apps for either the voter or election officials.  Those are downloaded separately.

#### ./ballot

Contains the ballot information for this GGO if there is ballot information.  For the national GGO for the 2018 election which does not contain a presidential contest, this would probably be empty except perhaps for a welcome message for the voter.

Each GGO contributes to the ballot that is presented to the voter at the precinct/Voting Center (VC).

#### ./votes

Votes are _collected_ in the votes folder.  The default configuration is that only GGO repos that have a precinct/VC will be collecting votes.  However it is configurable for a precinct/VC to commit votes to a parent GGO repo.  So, the root GGO repo in most cases would not directly contain any votes since it would not have a precinct/VC associated with it.

### 2.1) Child GGO's - Git Submodules

A child GGO is one where the VOTES Certificate Authority (CA) for this GGO creates an intermediate CA and allocates/associates sub GGO information with the ./info folder.  The git submodule tree looks like the following:

#### ./ggos/\<sub GGO class name>/\<instances name>

The \<sub GGO class name> is an arbitrary I18n name given to the specific intermediate CA's assigned by this CA.  For the US election this would be the 50 states plus any territory or other geographical geopolitical location.  (If voting is taking place electronically via the internet or other network, other or additional location coordinates will apply.)  This configuration info is also located in the ./info folder.

The following are examples.

#### ./ggos/state/Alabama, Alaska, Arizona, ...

The US national GGO will define 50 git submodules under the ggos subfolder.

#### ./ggos/town/Abbeville, Adamsville, Addison, ...

Each state entity (specifically each sub GGO who has been delegated as an intermediate certificate authority) can independently select its sub GGO class name for their state.  As an example Alabama may use the I18n string _town_ in its VOTES repo as above.

### 2.2) Multiple / Parallel GGOs

It is possible to have multiple and different classes of sub GGOs. For a state with both towns, counties, boroughs, congressional districts, school districts etc that can overlap in effectively arbitrary ways.  In the VOTES framework this is implemented as multiple/sibling \<sub GGO class names>:

#### ./ggos/county/Alameda, Alpine, Amador, ...
#### ./ggos/town/Adelanto, Agoura Hills, Alameda, ...

The above may be how California decides to handle its counties and towns.

Regardless of the number of different sub GGO class names, the __info__ section will determine how the sub GGO's are handled.  In addition, though a state, county, or town repo can be cloned in isolation, doing so will not result in a valid VOTES clone.  A valid VOTES clone of the election will include the repo hierarchy from the root repo, including all sibling GGO's in the hierarchy.  In this manner a California precinct/VC repo will also contain the town and county submodule/subtree repo trees and information.

In the California case, each town will in fact get a copy of those counties to which it has an overlay with.  A specific town may reside within multiple counties and a county will usually contain multiple towns.  It is configurable via the __info__ folder whether in this case California defines how the two GGO's overlay (which towns are in which county and vice versa) or if the counties or towns decide that.  Technical note - California actually owns the authority to decide but can pass the authority to either the county or town to define that information.

Regardless of delegation of the definition, each town and county clone only the relevant upstream and sibling repos.  At some level in this hierarchical tree, a GGO will want to actually collect votes on election day.  Actual votes are collected in the votes folder but require the repo to be configured as such, again via data in the ./info section.

As an example, the following section assumes the town of Cambridge Massachusetts.  When the city of Cambridge clones the VOTES repo for the 2018 election, assuming that all the parent GGO are indeed participating in a VOTES framework, the directory tree will look someting like:

```
US-2018-National-Election/.git
                          info
                          ballot
                          bin
                          votes
                          ggos/Massachusetts/.git
                                             info
                                             ballot
                                             bin
                                             votes
                                             ggos/Cambridge/.git
                                                            info
                                                            ballot
                                                            bin
                                                            votes
                                                            ggos/5th Congressional District
                                                                 7th Congressional District
                                                                 ward 1-1
                                                                 ward 1-2
                                                                 ...
                                                                 ward 11-2
                                                                 ward 11-3
```
Each Congressional District and Ward would have a git repo where there respective ballot contests can be entered.  When the voter gets a ballot from the VC (either an absentee, early, or on election day ballot), the VOTES framework will generate the correct ballot for the address.

Implementation note: it is possible for precincts/VC to share the same repo or leverage a parent repo to cast votes into - it is configurable into which repo a precinct/VC submits votes to.

# 3) How Ballots Are Entered

A paper or electronic ballot, generated by the VOTES framework, is custom generated for the address of the identified voter.  The voter fills out the ballot and submits it to a VOTES compatible Vote-capture device nominally operated by an election official at a precinct/VC.  The ballot is verified not to contain any overvotes or undervotes.  An overvote will block the submittal of a ballot while an undervote is easily override-able by the election official, in theory after validating with the voter that the undervote was intentional.

Once validated by the VOTES framework, the ballot is submitted to the git repo and the commit digest is returned to the voter.  A copy of the digest is also placed on either the paper or electronic ballot.  In the case of the paper ballot, the precinct/VC retains the physical copy of the ballot and associated digest as an air gapped record.  In the case of an electronic ballot, the submitted electronic ballot printed and stamped and retained by the precinct/VC as an air-gapped copy.

Implementation note - a precinct/VC can independently decide to go digital and employ a separate computer system and network to retain copies of the voters ballots.

Note - during pre-election (before early ballots are accepted due in part to the fact that the ballot is not yet complete), the VOTES repos are read-write accessible via the cert chain referenced above and casting votes is not possible.

# 4) What Is Accessible When

During the pre-election, the authoritative GGO entities create their respective GGO ballot.  They can also run test election day trials to test their ballots, their selected tally algorithms, etc.  Once all the GGO'S have completed and tested their ballots, early voting can commence.

## 4.1) Early Voting

Precincts/VC can choose whether they enter early-ballots when received or on the day of election.  The digests associated with the ballot can be made available to the early voter as the precinct sees fit, for example either by physical post or electronically.

Voters can validate their ballot at any time by entering their digest to the VOTES SaaS framework.  Importantly the election repos themselves __are not__ accessible.  They have been isolated from the internet.  The VOTES SaaS framework contains applications that given a specific ballot digest (protected with capta's etc so that it is not possible to collect a copy of the votes thus far cast), will return the ballot of interest.

## 4.2) Election Day

Voters physically enter a precinct/VC and cast paper ballots.  For precincts/VCs that wish to support electronic balloting, ballots are entered electronically.

## 4.3) Post Election Day

Once all polls close for all precincts/VCs associated with the election, read-only copies of the complete VOTES repo are made available.  At that point the repos are downloadable.  Since the repos already contain the tally algorithms for all contests, any interested party can perform the tally and determine the outcome.

In addition each voter can still validate that their ballot is part of any tally of interent.  Each precinct/VC can validate that the correct number of votes have been submitted.  Importantly any GGO or voter can inspect
any other GGO for irregularities or issues.

### 4.3.1) Ballot Nullification

Note that after the polls close, precincts/VCs can access their ballots, both the ballots entered in VOTES and the air-gapped physical ballots.  The latter may or may not contain voter identification such as name and address.  The former (the ballot in the VOTES framework) __does not__ contain voter identification.

Any authority in the cert chain can decide to subsequently decide to nullify a vote.  If a ballot is to be nullified, the authorized entity must commit to the appropriate git repo a ballot nullification which will remove the ballot from any tallies it is a part of.  However the history of the ballot stays intact.

The precinct/VC may decide to contact the voter via its air-gapped ballot copy that their vote has been nullifies.  However if the voter checks the VOTES SaaS or clones a copy of the election, they would directly see who, why, when, and where their vote was nullified.  They can directly contact the authorized entity that nullified the ballot or appeal to a higher authority in the cert chain.  Depending on the outcome of the appeal, the nullification itself can be nullified.

Thus, the tally may change after the polls close depending on the circumstances.

But in any case, the latest clone of the VOTES framework for the election will have the latest tally for all contests.
