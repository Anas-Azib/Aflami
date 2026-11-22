# Aflami hosting plan and risks

This is the hosting and risks part of the revised proposal. It says where Aflami version 1 runs, what each free plan allows, and what could stop the project, with how we prevent it and what we do if it happens. It will be merged into the revised proposal in week 13. The reasons behind the hosting choices are design decisions DD-05 to DD-10 in the [design document](../design/Aflami_SDD.md).

| Field | Value |
| --- | --- |
| Course | CPIT-498 (design), built in CPIT-499 |
| Authors | Anas Aazib, Abdullah Al-Jadani |
| Supervisor | Prof. Adeeb Rida Noor |
| Checked against | The providers' own pricing and limits pages, November 2026 |

## 1 Hosting plan

Version 1 costs nothing to host. Every service below has a free plan that covers the build and the evaluation with two teams; the only item paid by use is the AI model, and AI drafts are a Should feature.

| Service | Plan | What runs there | Free limits that matter to us | Why the limits are enough |
| --- | --- | --- | --- | --- |
| Vercel | Hobby, free | `aflami-web` (Next.js), `aflami-api` (NestJS, one function), the AI drafts job (Should) | 1 million function calls, 4 hours of active CPU and 100 GB of data transfer a month; a request stops after 5 minutes and carries at most 4.5 MB | Short-film teams are small, and the heavy traffic, the video, goes to YouTube. Files bypass Vercel through signed links (DD-07), and AI drafts run as a background job (DD-09) |
| Supabase | Free | The PostgreSQL database and the file storage | 500 MB of database, 1 GB of files, 5 GB of data transfer a month; pauses after one week without activity; no automatic backups | The 18 tables hold text and take a few MB. Files are capped at 10 MB each and cleaned by the retention rules (SRS 3.5). The daily job prevents the pause and makes our own backup |
| GitHub | Free | The public repository; GitHub Actions for the tests and the daily jobs; the private `aflami-backups` repository | Actions are free for public repositories; scheduled workflows stop after 60 days without repository activity | The tests and the daily jobs take minutes a day. The 60-day rule is risk R-05 below |
| YouTube | Free | The films, shown through the embedded player | — | — |
| AI model (Should) | Paid by use | Drafts from a script, called only from the back end | Price per use, set by the provider we choose | A daily limit of drafts per project keeps the cost known (R-10) |

**If a free plan is outgrown.** Supabase Pro costs $25 a month and adds daily backups kept for 7 days, with no pausing. Vercel Pro costs $20 a month per developer seat. Neither is needed for version 1; R-16 says when we would use them.

### 1.1 Setup checklist

Each step below avoids a problem we already know about, so it is done once, when the hosting is first set up, and checked again before the evaluation.

1. Create the Supabase project and both Vercel projects in the same region, so each database query stays inside one region.
2. Connect the API through Supabase's pooler in transaction mode (port 6543) and turn prepared statements off in the database library, because the pooler does not support them in that mode.
3. Keep every password and key in Vercel's and GitHub's settings only, and list `.env` files in `.gitignore` (NFR-SEC-07).
4. Keep the repository public and on Anas's personal GitHub account. Vercel's free plan deploys commits by other people (Abdullah) only for public repositories, and cannot connect a repository owned by a GitHub organization.
5. Store the backup key in GitHub's secret settings and give each student an offline copy.
6. Restore one export from `aflami-backups` into an empty database, once, to prove the backups work.

## 2 Risks

Likelihood and impact are Low, Medium or High. Priority comes from both: we score them 1, 2 and 3 and multiply; 6 or more is High, 3 or 4 is Medium, 2 or less is Low. The table is sorted by priority. We review it in every weekly update, and the owner of a risk that happens reports it there.

| ID | Risk | Likelihood | Impact | Priority | How we prevent it | If it happens | Owner |
| --- | --- | --- | --- | --- | --- | --- | --- |
| R-01 | The build takes longer than CPIT-499 allows | High | High | High | Build the whole Must journey first, thinly, from creating a project to publishing the film page, then deepen it. Keep the scope sheet's Must, Should and Future levels, and track the weekly plan in every update to the supervisor | Build the Should features last, in the order of the scope sheet, and report what moved to future work | Both |
| R-02 | Setting up the stack takes longer than planned: NestJS as a Vercel function, the Supabase pooler, signed uploads | Medium | High | High | The small permission prototype in week 14 runs on the real stack and the real hosting; the setup checklist (1.1) covers the known traps | Ask the supervisor early, and start from Vercel's and Supabase's official examples | Anas |
| R-03 | A permission bug lets a person see or change another project's data, such as its script or applications | Medium | High | High | One permission check that refuses by default (DD-03); automated permission tests on every pull request (NFR-MNT-01); unique rules in the database (DD-19) | Fix and redeploy the same day, read the activity log to see what was reached, and tell the affected teams | Anas |
| R-04 | One student is unavailable for weeks (illness, exams) | Medium | High | High | Every module has an owner, but the other student reviews each pull request, and the design document describes every module | The other student takes over that module from the documents, and the scope is cut as in R-01 | Both |
| R-05 | GitHub stops the daily workflow after 60 days without repository activity, so the backups and the keep-alive stop | Medium | High | High | Commit at least once a month during breaks, and check the workflow page in every weekly update | Turn the workflow back on and run it by hand; if Supabase paused meanwhile, see R-08 | Anas |
| R-06 | We cannot recruit two real filmmaker teams for the evaluation, or a team stops halfway | Medium | High | High | Contact teams in week 13, keep one more team in reserve, and give each team a short first session | Evaluate with the teams we have, and say so plainly in the final report | Abdullah |
| R-07 | Work is overwritten when both students edit the same file (it happened to the SRS in week 12) | Medium | Medium | Medium | Pull before editing, work on branches with pull requests, and keep one owner per document section | Restore the lost part from the git history | Both |
| R-08 | The Supabase project pauses after a week without activity | Medium | Medium | Medium | The daily job uses the database every day | Resume it from the Supabase dashboard; a paused project keeps its data for up to a year | Anas |
| R-09 | The Arabic right-to-left layout breaks on some phones, or with Arabic and English mixed in one line | Medium | Medium | Medium | Build right to left from the first screen, and test on Android and iPhone with Arabic sample data before the evaluation (NFR-PRT-01) | Fix it before the evaluation; the evaluation checklist lists each screen | Both |
| R-10 | AI drafts (Should) are poor, or the model costs more than expected | Medium | Medium | Medium | A person approves every draft; we measure the acceptance against the targets in the scope sheet; a daily limit of drafts per project | Move to another model, or lower the daily limit | Both |
| R-11 | Data is lost: a wrong delete, a bad migration, or a provider failure | Low | High | Medium | The daily encrypted export to `aflami-backups` keeps 30 days; every migration is reviewed in a pull request; one test restore (1.1, step 6) | Restore the latest export | Anas |
| R-12 | The backup key is lost or leaked | Low | High | Medium | The key lives in GitHub's secret settings, with an offline copy for each student, and never in the repository | Lost: make a new key and a new export the same day. Leaked: make a new key and delete the old exports | Anas |
| R-13 | A password or key is pushed to the public repository | Low | High | Medium | Secrets only in Vercel and GitHub settings (NFR-SEC-07), `.env` in `.gitignore`, and GitHub's secret scanning | Change the key at once, then remove it from the history | Both |
| R-14 | A script sent to the AI model leaks or is used to train the model | Low | High | Medium | The owner agrees before any script is sent; calls go from the server only; we choose a provider whose terms exclude training on our data; scripts are deleted on request | Stop sending scripts, tell the owners, and report it to the supervisor | Anas |
| R-15 | The repository becomes private or moves to a GitHub organization, and Vercel's free plan stops deploying Abdullah's commits | Low | Medium | Low | Keep it public on Anas's account (1.1, step 4) | Make it public again, or move to Vercel Pro ($20 a month) | Anas |
| R-16 | A free plan limit is reached, or a provider changes its free plan | Low | Medium | Low | Retention rules, the 10 MB file cap and video on YouTube keep usage low; we check usage monthly; standard tools (PostgreSQL, Node.js) let us move | Delete test data, or pay for Supabase Pro ($25) for the evaluation month; if the terms change, move using the latest export | Anas |
| R-17 | A film's YouTube video is removed, made private, or does not allow embedding | Medium | Low | Low | The link is checked when the page is published (UC-11), and the help page tells owners to use a public or unlisted video that allows embedding | The film page says the video is not available, and the owner updates the link | Abdullah |

Covers: NFR-REL-01, NFR-SEC-07, NFR-PRV-06, NFR-MNT-01, NFR-PRT-01.

## Sources

- [Vercel pricing](https://vercel.com/pricing) and [Vercel limits](https://vercel.com/docs/limits)
- [Vercel Functions limits](https://vercel.com/docs/functions/limitations)
- [Vercel: project collaboration on the Hobby plan](https://vercel.com/docs/deployments/troubleshoot-project-collaboration)
- [Supabase pricing](https://supabase.com/pricing) and [Supabase project pausing](https://supabase.com/docs/guides/platform/free-project-pausing)
- [Supabase: disabling prepared statements](https://supabase.com/docs/guides/troubleshooting/disabling-prepared-statements-qL8lEL)
- [GitHub Actions billing](https://docs.github.com/en/actions/reference/usage-limits-billing-and-administration) and [disabling and enabling workflows](https://docs.github.com/en/actions/how-tos/manage-workflow-runs/disable-and-enable-workflows)
