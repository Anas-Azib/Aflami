# Aflami permission prototype: plan

Prof. Adeeb recommended a very small prototype that checks permissions, to find problems early (plan task 32). This plan says what the prototype must prove, what it builds, how it is tested and what it leaves in the repository. It builds one piece of the real design, sequence diagram SD-4 and its rules table (design document, section 6), on the real stack and the real hosting.

| Field | Value |
| --- | --- |
| Owner | Anas |
| Plan task | 32, optional, two working days |
| Based on | SD-4 and its rules table; DD-03 (one check that refuses by default); hosting setup checklist (hosting plan, section 1.1) |
| Requirements | FR-ACC-02, FR-ACC-03, FR-FILM-02, FR-FILM-03, NFR-MNT-01 |
| Risks it reduces | R-02 (setting up the stack takes longer than planned), R-03 (a permission bug exposes another project's data) |

## 1 What it must prove

1. **The rules table works as code.** Every action is allowed or refused exactly as SD-4's table says, an action with no rule is refused (DD-03), and a permission comes only from memberships, never from `projects.created_by`.
2. **The stack works together on the real hosting.** NestJS runs as one Vercel Function and reaches Supabase PostgreSQL through the pooler in transaction mode, as the setup checklist describes.
3. **A broken rule cannot reach `main`.** The tests run on every pull request, and a failing test blocks the merge (NFR-MNT-01).

## 2 What it builds

**In the prototype:**

- Five tables, taken from the ERD as they are: `users`, `projects`, `memberships`, `member_roles` and `casting_calls`, with the unique rule on (user, project) in `memberships`.
- `PermissionGuard.canAct(actor, project, action)`, with the rules table kept as data in one place, so a reviewer reads the rules once.
- One endpoint for each action in the test table (section 3). Each one only asks the guard and answers allowed or refused; nothing else happens.
- Sign-in for the seeded test users only. It gives a signed session, and the API reads the actor from it, as in SD-4 step 1; a visitor has no session.
- Seed data: three projects (P, a Draft, and Q, Published, with the same team; R, a Draft of another team), an Open casting call in P, and eight test users with `example.com` emails. No real person's data.

**Left out:** every screen, sign-up, file uploads (only the permission to get an upload link), notifications, the activity log and the AI drafts.

## 3 The test table

Each cell is one automated test: Yes means the guard allows the action, No means it refuses it. The first row opens Q; every other row acts on P. The owner of P and Q is the Owner column; their `created_by` points to K, who is only a member, to prove the guard never reads that column.

| Action (rule level) | Visitor | Viewer | Creator, not in P | Member | Member with the Screenwriter role | Manager | Owner | Former manager (LEFT) | K: in `created_by`, only a member |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Open Q's published film page (Public) | Yes | Yes | Yes | Yes | Yes | Yes | Yes | Yes | Yes |
| Open the Draft project P (Member) | No | No | No | Yes | Yes | Yes | Yes | No | Yes |
| Apply to P's Open casting call (Creator) | No | No | Yes | Yes | Yes | Yes | Yes | Yes | Yes |
| Get a link to upload the script (Screenwriter) | No | No | No | No | Yes | Yes | Yes | No | No |
| Edit the film details (Manager) | No | No | No | No | No | Yes | Yes | No | No |
| Publish the film page (Owner) | No | No | No | No | No | No | Yes | No | No |
| Give manager rights (Owner) | No | No | No | No | No | No | Yes | No | No |
| An action with no rule | No | No | No | No | No | No | No | No | No |

That is 72 tests. Four more check what the table cannot show:

- The manager of P is refused when editing R, which belongs to another team (no permission leaks between projects).
- A request for a project that does not exist is refused, the same way as a forbidden one.
- The database refuses a second membership for the same person and project.
- When Q is hidden, the visitor is refused its page and a member still opens it (FR-FILM-02).

## 4 How it runs

- **Locally:** the tests run against a test database with the seed data.
- **GitHub Actions:** the tests run on every pull request, and branch protection on `main` requires them to pass.
- **Vercel and Supabase:** the API is deployed as one function in the same region as the Supabase project, connected through the pooler on port 6543 with prepared statements off (setup checklist, steps 1 and 2). Every key is kept in Vercel's and GitHub's settings only, and `.gitignore` lists `.env` files (step 3, NFR-SEC-07).
- **The database library** is chosen while building the prototype and recorded in the design document as DD-25, with why it was chosen and what was not.

## 5 Steps

The prototype is time-boxed to two working days. It is optional (plan task 32), so it never delays the final presentation.

1. Create the Supabase project and the Vercel project in one region, following the setup checklist.
2. Start the NestJS API with the five tables and the seed data, and deploy it once.
3. Write `PermissionGuard`, the rules table and the endpoints.
4. Write the 76 tests, run them in GitHub Actions, and turn on branch protection.
5. Call the live URL once with an allowed request and once with a refused one, and write the result.

## 6 Done when

- All 76 tests pass locally and in GitHub Actions, and changing one rule on purpose makes a test fail and blocks the merge.
- On the live Vercel URL, an allowed request and a refused request each give the expected answer, through the pooler.
- The result is written down: the test report, how long the setup took, the time of the first request and of a later one, and every setup problem met, each added to the setup checklist.

## 7 What it leaves in the repository

- `prototype/permission-check/`: the code, a README that says how to run it, and `RESULT.md` with the result.
- Updates to the setup checklist (hosting plan 1.1), DD-25 in the design document, and R-02 and R-03 reviewed in the next weekly update.
- The guard, the rules table and the tests carry into CPIT-499 weeks 1 to 4 (proposal, section 6.2); the rest of the prototype is thrown away.
