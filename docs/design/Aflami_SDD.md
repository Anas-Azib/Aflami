# Aflami Software Design Document
 
## Document control
 
This document shows how Aflami version 1 will be built. The [Aflami Software Requirements Specification](../requirements/Aflami_SRS.md) says what it must do, including the use case diagram; this document holds every other diagram the supervisor asked for.
 
| Field | Value |
| --- | --- |
| Course | CPIT-498 (design), built in CPIT-499 |
| Authors | Anas Aazib, Abdullah Al-Jadani |
| Supervisor | Prof. Adeeb Rida Noor |
| Version | 0.1, template |
 
**Rules for every diagram:**
 
- Draw it in one tool (draw.io or PlantUML), keep its source in `design/diagrams/`, export the image to `design/exports/`, and place the image in its section here.
- Under each diagram, list the requirement IDs it covers (FR-…, NFR-…).
- Use the same names as the SRS: table names, state names and role names (Owner, Manager, Member, Applicant, Viewer).
- The content in each section below is *starter* material from the design so far; replace each *Guidance* line with the diagram and delete it before you submit.
## 1 Architecture: components and deployment
 
Aflami is one web app and one back end with a module per feature, over one PostgreSQL database on Supabase, which also stores the files; the film video stays on YouTube.
 
| Component | Technology | Responsibility | Talks to |
| --- | --- | --- | --- |
| Web app | Next.js, React, TypeScript | Arabic screens, forms, the public film page | Back-end API |
| Back-end API | NestJS | Modules: Accounts and roles, Projects and scripts, Casting and team, Approvals, Tasks, Film page and credits. A permission guard runs on every request | Database |
| Database | PostgreSQL on Supabase (free plan) | The tables in section 2, reached through Supabase's connection pooler in transaction mode | Back-end API only |
| File storage | Supabase Storage, private buckets | Scripts, task attachments and photos. The browser uploads straight to storage with a signed link; private files open through links that expire | Browser, back-end API |
| Video | YouTube embedded player | Plays the film on the film page | Viewer's browser |
| Build and deploy | GitHub Actions, Vercel | Runs the tests on every pull request, then deploys | Repository |
| Scheduled jobs | GitHub Actions | Once a day: an encrypted database export kept outside the repository, and the cleanup rules of SRS 3.5. Also keeps the free Supabase project active | Back-end API, Supabase |
| AI drafts (Should) | Background job (Vercel Workflows) calling an external AI model from the server | Reads an episode's script and saves drafts for a person to approve; the page shows progress, and the user can leave and come back | Back-end API, AI model |
 
*Guidance: draw two diagrams. (1) A UML component diagram of the six modules and what each one calls, so the module links are visible. (2) A deployment diagram with these nodes: the user's browser, Vercel, Supabase (database and files), YouTube and GitHub.*
 
*Decided in week 12: Supabase hosts the database and the files. AI drafts, if built, run as a background job with no time limit, because a normal Vercel request stops after 5 minutes.*
 
Covers: FR-ACC-02, NFR-MNT-01, NFR-PRT-01, NFR-REL-01, NFR-SEC-05, NFR-PRV-06.
 
## 2 Data model (ERD)
 
Eighteen tables hold version 1: seventeen are needed by Must features, and one (reactions) only by a Should feature. The key idea is one chain: an accepted application creates a membership, the membership owns tasks, and it becomes a credit. Every project also has episodes: a film has exactly one, which the site never calls an episode, and a series has many. Series pages can then be added later without changing the design.
 
![Aflami ERD](../../design/exports/Aflami_ERD.png)
 
Source: `design/diagrams/Aflami_ERD.drawio`.
 
| Table | Holds | Links to | Rules |
| --- | --- | --- | --- |
| users | One account: name, email, password hash, account type (viewer or creator) | — | Email is unique |
| user_settings | One user's settings: language, theme, AI suggestions on or off, when the "next episode" pop-up appears | users | Exactly one row per user, created with default values at sign-up. Visitors without an account keep their language and theme in a browser cookie. The pop-up timing is stored now and used only when Aflami plays its own videos (Future) |
| projects | One film or series: title, kind (film or series), idea, synopsis, genre, year, language, status (draft, published or hidden) | users (creator) | Only the owner publishes or hides it |
| episodes | One video of the project: season, number, title, YouTube link, duration, status (draft or published) | projects | A film has exactly one; a series has many, numbered within each season. Series pages are a Should feature |
| scripts | The script file of one episode and its type (PDF, DOCX, Pages or TXT) | episodes | At most one per episode, so a film has one: a new upload replaces the old file; only members can open it |
| memberships | A person's place in a project: permission (owner, manager or member) and status (active or left) | users, projects | One per user and project; one owner per project, many managers allowed; never deleted |
| member_roles | The film roles a member holds, with the character name for actors | memberships | A member can hold many film roles |
| invitations | An invitation to a registered user, with the film role offered | projects, users | Sent by the owner or a manager; accepting creates a membership |
| casting_calls | A public call for one film role, with a deadline | projects, episodes (optional) | Status open or closed; closing stops new applications only. The episode is set only for a role in one episode |
| applications | One person's application: message, optional link and photo, shortlisted flag | casting_calls, users | One per person and call; waiting applications stay when the call closes |
| tasks | A piece of work: owner, status, deadline, and optionally its episode | projects, memberships, episodes (optional) | Owner becomes empty when that member leaves; the owner or a manager can reopen a done task |
| task_attachments | Images, files or links on a task | tasks | Size limit per file |
| credits | One line of the credits | projects, memberships, episodes (optional) | Proposed from the team, then approved; membership is empty for people outside Aflami; an empty episode means the whole film or series |
| ratings | A creator's star rating of a film, 1 to 5 | users, projects | One per user and film; the film's own team cannot rate it |
| reactions (Should) | A like or dislike | users, projects | One per user and film; needs an account |
| notifications | A message to one user inside Aflami | users, projects | Read or unread |
| announcements | A message from the owner to the whole team: a title and the text | projects, users (author) | — |
| activity_log | Who did what and when in a project | projects, users (actor) | Written by the system, never edited |
 
*Fields ending in `_by`, and `author_id` and `actor_id`, record who did something. They link to users; the diagram leaves out their lines to stay readable.*
 
*Decided in weeks 11 and 12: invitations stay in version 1; the second permission level is called Manager; only the newest script is kept; an application is a message plus an optional link; closing a call leaves waiting applications as they are; shortlisting is a yes/no flag, not a separate status; a project is a film or a series, and every project has episodes; the owner or a manager can reopen a done task.*
 
Covers: SRS 3.5, FR-ACC-03, FR-PRJ-02, FR-CST-02, FR-TEAM-01, FR-TEAM-02, FR-FILM-01.
 
## 3 Class diagram
 
Three diagrams, in the same pattern as the use cases. The overview shows all eighteen entity classes, coloured by module, with their main attributes and how they relate. The two detail diagrams open up the most complex areas, with their services, every method the sequence diagrams in section 6 call, and the enumerations.
 
1. Overview: ![Aflami class diagram 1, overview](../../design/exports/Aflami_Class_Diagram.png)
2. Casting and team: ![Aflami class diagram 2, casting and team](../../design/exports/Aflami_Class_Casting_Team.png)
3. Tasks, credits and publishing: ![Aflami class diagram 3, tasks, credits and publishing](../../design/exports/Aflami_Class_Tasks_Credits.png)
Sources: `design/diagrams/Aflami_Class_Diagram.drawio`, `design/diagrams/Aflami_Class_Casting_Team.drawio`, `design/diagrams/Aflami_Class_Tasks_Credits.drawio`.
 
**Design rules.** Entity classes hold data; services hold the behaviour, one group of services per back-end module. Every service method that changes a project calls `PermissionGuard.canAct` first and refuses the request if it returns false (UC-22). Composition (a filled diamond) marks a part that cannot exist without its whole: an application without its casting call, a member role without its membership, an attachment without its task, an episode without its project, a script without its episode, and a user's settings without the user.
 
**Services and their methods.** Sequence diagrams use exactly these names. The table lists the methods that change data; simple reads (a list of my projects, a film page) are queries in the same service and are left out.
 
| Service | Module | Methods |
| --- | --- | --- |
| PermissionGuard | All modules | canAct(user, project, action): boolean |
| AccountService | Accounts | register(data): User; signIn(email, password): Session; signOut(session): void; changePassword(user, oldPassword, newPassword): void; becomeCreator(user): void; updateSettings(user, data): UserSettings |
| NotificationService | Accounts | notify(user, type, itemId): Notification; markRead(notification, user): void |
| ProjectService | Projects | create(owner, data): Project (also creates the film's single episode); update(project, data, actor): void; addEpisode(project, data, actor): Episode (series, Should); updateEpisode(episode, data, actor): void (details and YouTube link); publish(project, actor): void; setHidden(project, hidden, actor): void; missingForPublish(project): string[] |
| ScriptService | Projects | upload(episode, file, actor): Script (replaces the previous script) |
| FileService | Projects | upload(file): string; openLink(key): string (a link that expires) |
| ActivityLogService | Projects | record(project, actor, action, itemId): ActivityLog |
| CastingService | Casting and team | publishCall(project, data, actor): CastingCall; editCall(call, data, actor): void; closeCall(call, actor): void |
| ApplicationService | Casting and team | submit(call, applicant, data): Application; shortlist(app, actor): void; accept(app, actor): Membership; reject(app, actor): void |
| InvitationService | Casting and team | invite(project, email, filmRole, actor): Invitation; accept(inv, user): Membership; decline(inv, user): void |
| MembershipService | Casting and team | addMember(project, user, filmRole): Membership; addFilmRole(membership, filmRole, character): MemberRole; setPermission(membership, permission, actor): void; leave(membership): void |
| AnnouncementService | Casting and team | post(project, title, body, actor): Announcement (notifies every active member) |
| TaskService | Tasks | create(project, data, actor): Task; assign(task, owner, actor): void; updateStatus(task, status, user): void; reopen(task, actor): void; attach(task, item, user): TaskAttachment |
| CreditService | Film page | proposeFromTeam(project): Credit[]; edit(credit, data, actor): void; approve(credit, actor): void |
| RatingService | Film page | rate(project, user, stars): Rating; react(project, user, value): Reaction (Should) |
 
*The use cases' supporting steps map onto these methods: UC-17 Upload a file is `FileService.upload`, UC-20 Notify a user is `NotificationService.notify`, UC-21 Add a team member is `MembershipService.addMember`, and UC-22 Check permission is `PermissionGuard.canAct`.*
 
Covers: FR-ACC-02, FR-ACC-03, FR-CST-01, FR-CST-02, FR-TEAM-02, FR-TSK-01, FR-TSK-02, FR-FILM-01, FR-FILM-03.
 
## 4 Main workflow (activity diagram)
 
The activity diagram follows one film from the new project to the published page, with a swimlane for each actor: Owner or Manager, Applicant, Member, System, Viewer.
 
1. Owner creates the project and adds the script.
2. Owner or Manager publishes casting calls, and may invite known people.
3. Applicant applies.
4. **Decision:** Owner or Manager accepts or rejects. Accepted: the system creates the membership. Rejected: the applicant sees the result.
5. **Loop** back to step 2 until the team is complete.
6. Owner or Manager creates tasks and assigns an owner to each.
7. Members update their tasks. **Loop** until the work is done.
8. Owner adds the YouTube link.
9. System proposes the credits from the team's film roles.
10. Owner or Manager edits and approves the credits.
11. Owner publishes the film page; viewers can watch it.
*Guidance: draw steps 4, 5 and 7 as a decision and two loops, and steps 2 and 6 as work that can run at the same time.*
 
Covers: FR-PRJ-01, FR-CST-01, FR-CST-02, FR-TSK-01, FR-FILM-01, FR-FILM-02.
 
## 5 Record states (state diagrams)
 
Draw one state machine diagram per record; the supervisor asked for these at minimum: application, task and credit. Each transition says what triggers it and who may trigger it.
 
| Record | From | Event (who) | To | Effect |
| --- | --- | --- | --- | --- |
| Application | Submitted | Accept (Owner, Manager) | Accepted | Creates the membership, or adds the film role if already a member |
| Application | Submitted | Reject (Owner, Manager) | Rejected | Applicant sees the result |
| Task | To do | Start (task owner) | In progress | — |
| Task | In progress | Finish (task owner) | Done | — |
| Credit | Proposed | Approve (Owner, Manager) | Approved | Records who approved |
| Membership | Active | Leave (member, or Owner) | Left | Their open tasks lose their owner; their credits are flagged for review |
| Invitation | Pending | Accept or decline (invitee) | Accepted or Declined | Accept creates the membership |
| Project | Draft | Publish (Owner) | Published | Needs a YouTube link and approved credits |
| Project | Published | Hide or show (Owner) | Hidden, or back to Published | The page leaves or returns to public view |
| Casting call | Open | Close (Owner, Manager) | Closed | Stops new applications; Submitted applications stay as they are |
| Task | Done | Reopen (Owner, Manager) | In progress | — |
| Episode | Draft | Publish (Owner) | Published | A series can release one episode at a time; a film's only episode is published with the project |
 
*Note: the first presentation showed Published as a credit state. In this design, publishing belongs to the project, so slide 11 changes in week 14.*
 
*Decided in week 12: closing a call leaves Submitted applications as they are, and the owner or a manager can reopen a Done task.*
 
Covers: FR-CST-02, FR-TSK-01, FR-FILM-01, FR-FILM-02.
 
## 6 Key scenarios (sequence diagrams)
 
Four scenarios only, as the supervisor asked: the three flows that make Aflami different, plus the permission check that guards them all.
 
### SD-1 Accept an applicant (Abdullah)
 
Participants: Owner, Web app, ApplicationService, PermissionGuard, MembershipService, Database.
 
1. Owner chooses Accept on an application.
2. PermissionGuard checks that the Owner may decide in this project.
3. ApplicationService marks it Accepted and records who decided and when.
4. MembershipService creates the membership, or adds the film role if the applicant is already a member.
5. Web app shows the new member in the team.
Covers: FR-CST-02, FR-TEAM-01, FR-ACC-02.
 
### SD-2 Assign a task and update it (Abdullah)
 
Participants: Manager, Member, Web app, TaskService, PermissionGuard, Database.
 
1. Manager creates a task and assigns it to a member.
2. PermissionGuard checks the Manager's rights; TaskService saves the task as To do.
3. Member moves the task to In progress, then Done.
4. PermissionGuard checks that this member owns the task before each change.
Covers: FR-TSK-01, FR-ACC-02.
 
### SD-3 Approve the credits and publish the film page (Abdullah)
 
Participants: Owner, Web app, CreditService, ProjectService, Database, Viewer, YouTube.
 
1. CreditService proposes the credits from the team's film roles.
2. Owner edits and approves them.
3. ProjectService checks for a YouTube link and approved credits, then publishes.
4. A Viewer opens the page; the browser loads the video from YouTube.
Covers: FR-FILM-01, FR-FILM-02.
 
### SD-4 Permission check on any request (Anas)
 
Participants: User, Web app, API module, PermissionGuard, Database.
 
1. The user sends a request about a project.
2. PermissionGuard reads the user's membership in that project.
3. No active membership, or a permission too low: the request is refused and nothing changes.
4. Otherwise the module handles the request.
Covers: FR-ACC-02, NFR-PRV-01.
 
*Guidance: use the class and method names from section 3, and show the refusal path in each diagram as an alternative fragment.*
 
## 7 Diagram checklist and consistency checks
 
Seven diagram types make the full set the supervisor asked for; tick each when its image and its "Covers" line are in place.
 
- [x] Use case diagram, in the SRS section 3.2 (Anas, week 12)
- [x] ERD, section 2 (Anas, week 12)
- [ ] Component and deployment diagrams, section 1 (Anas, week 13)
- [x] Class diagram, section 3 (Anas, week 13)
- [ ] Activity diagram, section 4 (Abdullah, week 13)
- [ ] State diagrams for application, task and credit, section 5 (Abdullah, week 13)
- [ ] Sequence diagrams SD-1 to SD-3 (Abdullah) and SD-4 (Anas), section 6 (week 13)
**Before sending to the supervisor, check that the diagrams agree:**
 
- Every actor in the use case diagram is a role in section 1.5 of the SRS.
- Every class in a sequence diagram, and every method it calls, is in the class diagram.
- Every state in a state diagram is a value of a status field in the ERD.
- Every step of the activity diagram matches a use case.
- Every diagram's "Covers" IDs appear in the SRS traceability matrix (SRS 5.2).
 