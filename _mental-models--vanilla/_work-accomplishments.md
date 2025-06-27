# Work Accomplishments

## Promotion Path

- Focus on project leadership stuff (ticket sizing, ceremonies, sprint kick off, retro)

## 12/2/2024 - 12/24/2024

- PFS PDF report generation with balance sheet table splitting implemented
- Attempt to fix out of memory issue in CI build for cypress - able to identify issue is probably related to vite/rollup sourcemaps; added a script to log the heap memory information
- Optimized the index.html file for PDF service by injecting scripts only when not on the PDF service path
- PFS report service with enqueue method, Rails tagged logging, and LaunchDarkly feature flag integration
- Helped refactor component (ActionSelect) to use Aether Popover, which is more performant and less buggy
- PFS report service with enqueue method. Includes update to index.html (dynamic script loading) that should improve PDF generation performance slightly.
- VIMs (NCST, Estate Builder inputs)
- Wrote tickets and led sizing for My Notes Library M2
- Helped figure out an issue with failing PDF Cypress tests on staging. War room with George, Jason, Icaro, Ruokai, and others. Involved a bit of trial and error, looking at Datadog logs, using the rails console, and running the staging tests from local machine (pointing the local Cypress config to staging). We eventually figured out that it was due to missed assumptions regarding LaunchDarkly, the staging environment, and how Cypress specifically runs those PDF tests. Purple team to write COE (Correction of Errors).
- Waterfall test updates for advanced distributions (tile refactor) and funding option migration
- Balance Sheet bug fixes (FE React performance, sticky header/footer resizing, misc CSS)

## 11/4/2024 - 11/15/2024

- Worked on the Personal Financial Statement (PFS) TDD, including investigating the technical level of effort required, creating the required tickets, and leading the sizing sessions
- Got the `service-rendering-pdf-inline` working locally (which was not working before) and made the necessary code and README changes in a PR; Jason confirmed deployment to staging and production environments
- Started working on some of the Q4 performance tickets - investigations revealed suboptimal frontend code splitting, excess queries, and possibly slow queries (due to suboptimal indexes)
- Exploring Datadog

## ??

## 9/23/2024 - 10/11/2024

- Setup cypress to work with Launch Darkly feature flags (workaround)
- Copy step 2 to 3 event labels helper and query
- Misc VIM fixes (Estate Builder notes and fiduciaries, Organization Provision policy update for sample estates)
- Taxable sample estate updates for plan snapshot
- Fixed VIM for document copying related to non-converging sub-trusts
- Pair programming with Icaro to explain how step copying should work for irrevocable and joint revocable trusts
- Worked with Icaro to discuss approaches (both backend and frontend) for the balance sheet updates project - Includes updated account view, new asset view, and ability to specify percentage based asset type ownership for accounts of balance type total account
- Wrote TDD and tickets for the updates to balance sheet project (frontend focus) and led sizing

## 9/9/2024 - 9/20/2024

- Created document retention admin tool to view estate uploads for organizations with document retention period, can filter by organization or date range (on either created at or estimated deleted at)
- Wrote documentation for how to create Launch Darkly feature flags
- Documented how the provision notes works in the frontend in terms of the relevant query and mutation
- Pair programming with Icaro (initial backend work for Custom Notes Portal) - Added account ID to `OrganizationProvision`, updated default scope, added tests, and refactored allowed organization provisions query
- Migrated update notes action mutation to custom mutation with access to account ID; updated functionality of underlying provision notes service to handle creation of user library note
- Led ticket sizing for Copy Step 2 to 3 project (tickets created earlier)
- Created query for user notes
- Created mutations for creating, editing, and deleting user notes with extensive testing on the policy and functionality
- Implemented a workaround to enable launch darkly feature flag testing with Cypress e2e tests
