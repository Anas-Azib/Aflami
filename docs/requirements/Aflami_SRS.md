# Aflami Software Requirements Specification

## Document control

This is the requirements document for Aflami, structured after the SRS outline in ISO/IEC/IEEE 29148:2018. Sections 1 to 5 below follow that outline; replace each *Guidance* line with your own text and delete it before you submit.

| Field | Value |
| --- | --- |
| Project | Aflami: one platform for a short film's journey, from project to published film page |
| Course | CPIT-498 (requirements and design), Faculty of Computing and IT, King Abdulaziz University |
| Authors | Anas Aazib, Abdullah Al-Jadani |
| Supervisor | Prof. Adeeb Rida Noor |
| Standard | ISO/IEC/IEEE 29148:2018, Software requirements specification |
| Version | 0.1, template |

**Requirement IDs.** Every requirement gets an ID that never changes, so diagrams and tests can point to it:

| Prefix | Area | Writer |
| --- | --- | --- |
| FR-ACC | Accounts, roles and permissions | Anas |
| FR-PRJ | Projects and scripts | Anas |
| FR-CST | Casting calls and applications | Abdullah |
| FR-TEAM | Team, invitations and film roles | Abdullah |
| FR-TSK | Tasks | Abdullah |
| FR-FILM | Film page and credits | Abdullah |
| NFR-xxx | Non-functional (SEC, PRV, USA, PERF, REL, MNT) | Anas |

**Every requirement carries the same attributes**, as 29148 asks: ID, statement ("The system shall …"), rationale, priority (Must / Should / Future), source (user story or supervisor), and verification method (Test, Demonstration, Inspection or Analysis).

## 1 Introduction

### 1.1 Purpose

This SRS defines what version 1 of Aflami must do. It is the agreement between the team, the supervisor and the committee, and the base for the design in CPIT-498 and the build and tests in CPIT-499.

*Guidance: name the readers and how each one uses this document.*

### 1.2 Scope

Aflami is an Arabic-first web platform for student and independent short-film makers in Saudi Arabia and their crews. In one place, a team creates its project, casts its members, distributes the tasks and publishes a film page with full credits. Today these steps are spread across WhatsApp, shared drives and social media; Aflami links them into one record, so every person who worked on the film ends up in its credits. The first planned users are student filmmakers and university film clubs.

Aflami has three kinds of user. A **Visitor** has no account and can only watch published film pages. A **Viewer** has an account, which liking or disliking a film (a Should feature) needs. A **Creator** chooses that type at sign-up, or switches from Viewer later, and can create projects, apply to casting calls, join teams and rate films. Inside each project, a Creator is the Owner, a Manager or a Member (section 1.5).

Every feature has one of three levels:

- **Must:** part of version 1; the CPIT-499 build is not finished without it.
- **Should:** part of version 1 if time allows; built only after every Must feature works.
- **Future:** not in version 1.

| Area | Must | Should | Future |
| --- | --- | --- | --- |
| Accounts | Registration and sign-in; sign out; account type (Viewer or Creator) chosen at sign-up; permission levels Owner, Manager and Member, with the owner choosing managers; notifications inside the site; my projects page; change password using the old password | Profile with past credits and a link to past work; edit my profile; delete my account | Email notifications; reset a forgotten password (needs an email service); sign in with Google or Apple |
| Projects and scripts | Create a project and upload the script (PDF, Word, Pages or plain text; only the newest is kept); edit project details | Project overview; members download the script; archive or delete a project | Script version history; project poster; transfer ownership |
| Casting | Publish casting calls, apply, shortlist, accept or reject; a public link for each call; edit and close a call; applicant adds a photo or a demo-reel link | A page of all open calls; call details (location, shooting dates, paid or unpaid); my applications page; share buttons | Withdraw an application; filter and sort applications; automatic posting to social media; audition scheduling |
| Team | Invitations to registered users, accepted or declined on an invitation page; team page with each member's film roles; character names for actors | Leave a project; owner removes a member; change a member's film roles | Team contact list |
| Tasks | Create, assign and update tasks (To do, In progress, Done); reopen a done task; task attachments (images, files and links, with a size limit) | Overdue tasks highlighted; task comments; board view | Production scheduling; calendar view; my tasks across projects |
| Film page and credits | Credits proposed from the team, then edited and approved; film page with the YouTube video, synopsis, details and credits; only the owner publishes; public or hidden at any time; watching without an account; star rating 1 to 5 by Creators outside the film's team | Credits for people without an account; credits linked to profiles; public gallery of films; like or dislike | Our own video hosting or direct upload to YouTube; viewer search and recommendations; reorder credits; viewer comments |
| Communication | Announcements from the owner; activity log of who approved what, and when | Team chat | — |
| Interface and platform | Arabic, right to left, on phone and desktop; Arabic and English switch for the interface (team content is never translated); terms of use and privacy policy; help and FAQ | Light and dark theme; admin panel | Landing page; report inappropriate content |
| AI assistance | — | AI drafts of the roles and casting ad, the tasks, and the film title and description, from the script; a person approves, edits or rejects every draft; every suggestion and its outcome is logged; script consent and privacy before any script is sent | — |

**Success criterion.** Version 1 succeeds when two real filmmaker teams complete the whole journey on Aflami: create the project, build the team, finish the tasks and publish the film page. For each team we report the time per step, whether they completed it without help, and the errors they met.

### 1.3 Product perspective

Aflami is a new, self-contained web system. Today the same teams use WhatsApp, shared drives and social media for these steps; Aflami keeps the project, the people and their roles in one linked record.

| Interface (29148) | What Aflami needs (draft) |
| --- | --- |
| 1.3.1 System interfaces | YouTube embedded player on the film page |
| 1.3.2 User interfaces | Arabic-first, right-to-left, responsive web screens (list them; wireframes in the design document) |
| 1.3.3 Hardware interfaces | None: any device with a current web browser |
| 1.3.4 Software interfaces | PostgreSQL database; YouTube embed; GitHub Actions and Vercel for build and deployment |
| 1.3.5 Communications interfaces | HTTPS only. No email in version 1: invitations and results appear as notifications inside Aflami. |
| 1.3.6 Memory constraints | None specific. *Guidance: write "None" if that stays true.* |
| 1.3.7 Operations | Who deploys, how often the database is backed up, how a release is rolled back |
| 1.3.8 Site adaptation | Arabic locale and the Asia/Riyadh time zone for dates and deadlines |

### 1.4 Product functions

1. **Create a project:** the owner adds the idea and the script.
2. **Build the team:** the owner publishes casting calls, reviews applications and invites known people.
3. **Distribute the work:** tasks get an owner, a deadline and a status.
4. **Publish the film page:** the YouTube video and the credits, reviewed by the owner before publishing.

### 1.5 User characteristics

Every person has one account type. Inside each project, a Creator also has one permission level.

| User class | Who | What they can do | Expected skill |
| --- | --- | --- | --- |
| Visitor | Anyone without an account | Watch published film pages | None |
| Viewer | Account of type Viewer | Everything a Visitor can, plus like or dislike a film (Should) | Basic web use |
| Creator | Account of type Creator, chosen at sign-up or switched to later from Viewer | Create projects, apply to casting calls, join teams, and rate films 1 to 5 (not their own team's film) | Basic web use |
| Owner | Creator who creates the project | Everything, including publishing or hiding the film page and giving or removing manager rights | Basic web use |
| Manager | Member chosen by the owner; a project can have several | Everything except publishing and managing permissions: accept applicants, invite people, manage tasks, approve credits | Basic web use |
| Member | Accepted team member | See the project, update the tasks they own | Basic web use |
| Applicant | Creator who applies to a casting call | See only their own application until accepted | Basic web use |

*Decided in week 11: the second level is called Manager, so it never clashes with film roles such as Director or Producer. Decided in week 12: three account types, Visitor, Viewer and Creator; only Creators apply to casting calls and join teams.*

### 1.6 Limitations

- Two developers and one semester (CPIT-499) to build and test.
- Free hosting tiers, with their limits on requests and database size.
- Video playback depends on YouTube; Aflami stores links, not video files.

### 1.7 Assumptions and dependencies

- YouTube embedding stays available for public and unlisted videos.
- Two student teams agree to take part in the evaluation.
- *Guidance: add each assumption whose failure would change a requirement.*

### 1.8 Apportioning of requirements

Each requirement in section 3 takes the level of its feature in section 1.2: Must, Should or Future. CPIT-499 builds every Must requirement first, one area per iteration, then the Should requirements in the order the team agrees. Future requirements are listed for completeness and are not built or tested in version 1.

## 2 References

1. ISO/IEC/IEEE 29148:2018, *Systems and software engineering — Life cycle processes — Requirements engineering*.
2. Aflami project proposal, CPIT-498, revised version.
3. Aflami MVP journey document.
4. Aflami first presentation, November 2026.
5. [Aflami Software Design Document](../design/Aflami_SDD.md).

*Guidance: list every document this SRS depends on, with its version and date.*

## 3 Specific requirements

Each requirement below uses the attributes from Document control. The rows marked *starter* come from the design so far; keep, edit or delete them.

### 3.1 External interfaces

- **YouTube link:** the owner pastes a public or unlisted YouTube link; the system checks it is a YouTube video link before saving.
- *Guidance: for each input and output, give its source or destination, valid range and format.*

### 3.2 Functions

Functions are written three ways that point to each other: user stories (what people want), use cases (how the interaction runs) and requirements (what the system shall do, one testable sentence each).

**User story format, with one example:**

> **US-CST-02.** As an owner, I want to accept or reject each applicant, so that only the people I choose join my project.
>
> Acceptance criteria:
>
> - Given an application in Submitted, when the owner accepts it, then it becomes Accepted and the applicant becomes a member with the call's film role.
> - Given an applicant who is already a member, when accepted for a second call, then no second membership is made and the new film role is added.
> - Given a member without manager rights, when they try to accept, then the system refuses.

**Use cases to write:**

| ID | Use case | Primary actor | Writer |
| --- | --- | --- | --- |
| UC-01 | Register and sign in | Any user | Anas |
| UC-02 | Create a project and add the script | Owner | Anas |
| UC-03 | Give a member manager rights | Owner | Anas |
| UC-04 | Publish a casting call | Owner, Manager | Abdullah |
| UC-05 | Apply to a casting call | Applicant | Abdullah |
| UC-06 | Review an application | Owner, Manager | Abdullah |
| UC-07 | Invite a known person | Owner, Manager | Abdullah |
| UC-08 | Create and assign a task | Owner, Manager | Abdullah |
| UC-09 | Update a task's status | Task owner | Abdullah |
| UC-10 | Review and approve the credits | Owner, Manager | Abdullah |
| UC-11 | Publish the film page | Owner | Abdullah |
| UC-12 | Watch a film page | Viewer | Abdullah |

**Use case diagram:** *Guidance: one UML use case diagram with the five actors from section 1.5 and UC-01 to UC-12, placed here. Every other diagram is in the [Aflami Software Design Document](../design/Aflami_SDD.md).*

**Use case format, with one example:**

> **UC-06 Review an application.** Primary actor: Owner or Manager. Preconditions: signed in; the application is Submitted.
>
> Main flow:
>
> 1. The actor opens a casting call.
> 2. The system lists its applications.
> 3. The actor opens one and chooses Accept.
> 4. The system marks it Accepted, records who decided and when, and creates the membership with the call's film role.
> 5. The system shows the new member in the team.
>
> Alternative flow 3a: the actor chooses Reject; the system marks it Rejected and the applicant sees the result. Exception: the actor is not the owner or a manager; the system refuses and changes nothing. Postcondition: the applicant is a member of this project only. Related: FR-CST-02, FR-ACC-02.

**Functional requirements (starter):**

| ID | Requirement | Priority | Source | Verification |
| --- | --- | --- | --- | --- |
| FR-ACC-01 | The system shall let a person register with a name, email and password. | Must | UC-01 | Test |
| FR-ACC-02 | The system shall check the user's role in the project on every request that reads or changes project data. | Must | Supervisor, permissions design | Test |
| FR-ACC-03 | Only the owner shall give or remove manager rights; a project can have several managers. | Must | Week 11 decision, UC-03 | Test |
| FR-PRJ-01 | The system shall let a signed-in user create a project with a title, idea and script. | Must | UC-02 | Test |
| FR-PRJ-02 | The system shall take the script as a PDF, Word (.docx), Pages or plain-text file, show it only to project members, and keep only the newest version. | Must | Week 11 decision | Test |
| FR-CST-01 | The system shall let the owner or a manager publish a casting call with a film role, description and deadline. | Must | UC-04 | Test |
| FR-CST-02 | The system shall create a membership for the applicant when the owner or a manager accepts an application. | Must | US-CST-02, UC-06 | Test |
| FR-TEAM-01 | The system shall let one member hold more than one film role in a project. | Must | Table review scenario | Test |
| FR-TEAM-02 | The system shall let the owner or a manager invite a registered Aflami user; the invitation appears as a notification inside Aflami, and the user joins only after accepting. | Must | Week 11 decision, UC-07 | Test |
| FR-TSK-01 | The system shall move a task only through To do, In progress and Done. | Must | UC-09 | Test |
| FR-FILM-01 | The system shall propose credits from the team's film roles and let the owner or a manager edit and approve them before publishing. | Must | UC-10 | Demonstration |
| FR-FILM-02 | The system shall show a published film page to viewers without an account. | Must | UC-12 | Test |
| FR-FILM-03 | Only the owner shall publish the film page. | Must | Week 11 decision, UC-11 | Test |

### 3.3 Usability requirements

| ID | Requirement | Priority | Verification |
| --- | --- | --- | --- |
| NFR-USA-01 | The interface shall be in Arabic, right to left, on phone and desktop screens. | Must | Inspection |
| NFR-USA-02 | A first-time owner shall create a project and publish a casting call without help. | Should | Test with the evaluation teams |

*Guidance: state each usability requirement so the evaluation in CPIT-499 can measure it.*

### 3.4 Performance requirements

*Guidance: give numbers you can test, for example the response time for 95% of requests and the number of users at the same time. Choose them with the hosting plan; mark them as proposed until tested.*

### 3.5 Logical database requirements

- The data model holds: users, projects, scripts, memberships, member roles, invitations, casting calls, applications, tasks and credits (the ERD is in the [design document](../design/Aflami_SDD.md)).
- A user has at most one membership per project, and one application per casting call. A project keeps only its newest script: a new PDF upload deletes the old one.
- A member who leaves is marked as left, not deleted, so tasks and credits keep their history.
- *Guidance: add retention rules (how long data is kept) and who may read each table.*

### 3.6 Design constraints

- Web front end in Next.js, React and TypeScript; back end in NestJS; database in PostgreSQL.
- Hosting on Vercel; build and tests through GitHub Actions.
- Film videos stay on YouTube and are shown through its embedded player.
- Version 1 calls no external AI model.

### 3.7 Standards compliance

- ISO/IEC/IEEE 29148:2018 for this document.
- *Guidance: check what the Saudi Personal Data Protection Law asks for the personal data Aflami keeps (names, emails, scripts).*

### 3.8 Software system attributes

| ID | Attribute | Requirement (starter) | Verification |
| --- | --- | --- | --- |
| NFR-SEC-01 | Security | Passwords shall be stored only as salted hashes. | Inspection |
| NFR-PRV-01 | Privacy | A project's script shall be visible only to its accepted members. | Test |
| NFR-REL-01 | Reliability | The database shall be backed up at least once a day. | Inspection |
| NFR-AVL-01 | Availability | *Set a target that the hosting plan can meet.* | Analysis |
| NFR-MNT-01 | Maintainability | Automated tests shall run on every pull request before merge. | Inspection |
| NFR-PRT-01 | Portability | The system shall work on current Chrome, Safari and Firefox. | Test |

### 3.9 Supporting information

*Guidance: point to the wireframes, the glossary and any sample data that helps a reader understand the requirements.*

## 4 Verification

Every requirement in section 3 is checked in CPIT-499 by one of four methods: **Test** (run with set inputs and an expected result), **Demonstration** (shown working), **Inspection** (code or configuration reviewed) or **Analysis** (worked out from data or a model).

| Requirements | Method | When | Evidence |
| --- | --- | --- | --- |
| FR-ACC-02, NFR-PRV-01 | Test: automated permission tests | Every pull request | Test report in GitHub Actions |
| FR-CST, FR-TEAM, FR-TSK | Test: integration tests of each flow | Before the first integrated trial | Test report |
| FR-FILM-01, FR-FILM-02 | Demonstration | First integrated trial | Recording or checklist |
| NFR-USA-01, NFR-USA-02 | Test with the two evaluation teams | Evaluation phase | Time per step, completion without help, errors |
| NFR-SEC-01, NFR-MNT-01 | Inspection | Before the final report | Review checklist |

*Guidance: keep one row per requirement group, and make sure no requirement in section 3 is left without a row.*

## 5 Appendices

### 5.1 Acronyms and abbreviations

| Term | Meaning |
| --- | --- |
| SRS | Software requirements specification |
| MVP | Minimum viable product: the scope of version 1 |
| ERD | Entity relationship diagram |
| FR / NFR | Functional / non-functional requirement |
| US / UC | User story / use case |
| CI | Continuous integration: tests that run on every pull request |

### 5.2 Traceability matrix

This matrix ties every requirement to its story, use case, diagrams and test, so each diagram can be checked against the requirements.

| Requirement | User story | Use case | Diagrams | Test |
| --- | --- | --- | --- | --- |
| FR-CST-02 | US-CST-02 | UC-06 | Sequence: accept an applicant; State: application; ERD: applications, memberships | Integration test: accept creates one membership |

*Guidance: add one row per requirement as you write the diagrams in week 13.*
