# Contribution 1: 6.0: Table tree re-opens/animates everytime + forgets position when changing inner tab

**Contribution Number:** 1

**Student:** Ethan Reed

**Issue:** https://github.com/phpmyadmin/phpmyadmin/issues/18974

**Status:** Phase IV complete

---

## Why I Chose This Issue

I was interested in this issue because I wanted to try fixing a bug in a large codebase. I also noticed that the project has an active maintainer (and is  receiving frequent updates) so I thought it would be a good candidate for this course. I took this issue to explore PHP since I am not familiar with it, but I do have experience with web apps and web development so I felt it would be a good fit for me. I hope to learn how to navigate and debug issues in a real codebase, along with learning how to communicate with other people in a real-world project.

---

## Understanding the Issue

### Problem Description

The table tree in the UI always replays the tree opening animation, along with forgetting which page you were on (in a specific table tree) when clicking between tabs in the UI. This issue only comes into effect when browsing large tables (50+ entries), or with however entries are needed to cause the UI to display the table across multiple pages.

### Expected Behavior

- PHPMyAdmin should not re-open/animate the table tree after a "inner tab" has been selected.
- PHPMyAdmin should remember the previously selected "page" of the paginated table list after selecting a different "inner tab".

### Current Behavior

- PHPMyAdmin re-animates the table tree whenever a "inner tab" has been selected.
- PHPMyAdmin forgets the previously selected "page" of the paginated table list (resetting back to page 1) after selecting a different "inner tab".

### Affected Components

This appears to be a UI issue, so the parts of the codebase that are involved should be anything under the 'templates' folder (resources/templates), along with anything else that is related to the UI. 

---

## Reproduction Process

### Environment Setup

I set up phpMyAdmin locally using the project's built-in development server, along with a local MySQL instance. The setup process was pretty straight forward since the project moved to using php's 'composer' to manage dependencies. I then setup a docker container to run a MySQL instance locally (which is needed for phpMyAdmin since it is just a GUI). I also used Docker to pull phpMyAdmin 5.2.3 to verify behavior on an older version.

### Steps to Reproduce

1. Open phpMyAdmin and navigate to a database that has enough tables to trigger pagination in the navigation tree (in my case it was more than 50 tables).
2. Expand the database in the navigation tree and navigate to a page other than page 1 in the table list.
3. Click on any table, then switch between the inner tabs (Structure, SQL, Search, etc.).
4. Observe that the navigation tree replays the opening animation and the table list pagination resets back to page 1.

### Reproduction Evidence

- **My findings:** The bug was introduced when phpMyAdmin switched from AJAX-based navigation to full page reloads (commits 36a77f7 and 38b796f). phpMyAdmin already had a sessionStorage-based system for saving and restoring tree state, but it was silently broken due to a type mismatch in the server id comparison. This meant the restoration branch never ran, and every page load treated the tree as a fresh load.

---

## Solution Approach

### Analysis

I started by looking at the navigation code in `resources/js/modules/navigation.ts` and `resources/js/modules/navigation/event-loader.ts`. The event-loader has three branches on page load: fresh load (no saved state), restoration (server and token match), and different user (fallback). By adding console.log statements at each branch, I discovered it was always hitting the "different user" branch even when browsing as the same user on the same server.

The root cause turned out to be a strict equality comparison (`===`) between `CommonParams.get('server')` (an int, since PHP's `Current::$server` is typed as `int`) and `storage.server` (a string, since sessionStorage coerces all values to strings). The comparison `1 === "1"` always fails in JavaScript, so the tree state was never restored.

With the restoration fixed, I noticed the tree opening animation was still replaying on every page load. This was because `expandTreeNode()` always uses `slideDown('fast')` to animate the tree opening, even when restoring from saved state.

### Proposed Solution

Two changes:

1. Cast the server id to a string before comparing it in the event-loader, so the sessionStorage restoration branch actually runs.
2. Add a `skipTreeAnimation` parameter to `expandTreeNode()` so the animation can be skipped when restoring saved state, while still playing when the user presses the expansion button.

### Implementation Plan

Using UMPIRE framework (adapted):

**Understand:** The navigation tree's state persistence system exists but is broken due to a type mismatch, and the tree opening animation replays unnecessarily during state restoration.

**Match:** The codebase already has the sessionStorage save/restore pattern in place (`treeStateUpdate`, `traverseForPaths`, `filterStateRestore`). I followed the same patterns for my changes rather than introducing new ones.

**Plan:**
1. Fix the type mismatch in `event-loader.ts` by wrapping `CommonParams.get('server')` in `String()`.
2. Add `skipTreeAnimation` parameter to `expandTreeNode` in `navigation.ts`, defaulting to `false`.
3. Thread `skipTreeAnimation` through `showCurrent` and `fullExpand` so it reaches `expandTreeNode`.
4. Pass `paths !== null` as the `skipTreeAnimation` value in `requestNaviReload`, so it's only true during sessionStorage restoration.

**Implement:** Branch `fix/nav-tree-ui-fix` on fork https://github.com/etchre/phpmyadmin

**Review:** Ran `yarn eslint` on both changed files (passed clean). Checked that the code follows the existing comment and naming conventions in the codebase. Both commits include `Signed-off-by` as required by the project's DCO.

**Evaluate:** Manually tested by navigating between inner tabs, verifying the tree no longer re-animates and the pagination position is preserved. Also verified that manually clicking to expand/collapse tables still plays the animation as expected.

---

## Testing Strategy

### Manual Testing

- Verified tree state (open nodes and pagination) persists across tab switches.
- Verified the tree opening animation is skipped during page-load restoration.
- Verified the tree opening animation still plays when manually expanding a node by clicking.
- Verified behavior on a fresh session (no saved state) still works as expected.
- Used Docker to run phpMyAdmin 5.2.3 to confirm that the type mismatch bug also exists in older versions (it does, but the restoration worked there because navigation used AJAX instead of full page reloads, so the bug was masked).

---

## Implementation Notes

### Code Changes

- **Files modified:**
  - `resources/js/modules/navigation.ts` — Added `skipTreeAnimation` parameter to `expandTreeNode`, threaded through `showCurrent` and `fullExpand`. Used `paths !== null` in `requestNaviReload` to activate animation skipping during restoration.
  - `resources/js/modules/navigation/event-loader.ts` — Wrapped `CommonParams.get('server')` in `String()` to fix the type mismatch in the sessionStorage restoration check.

- **Key commits:**
  - `351993d` — Fix navigation tree state never being restored from sessionStorage
  - `d01fe03` — Allow skipping the tree opening animation in expandTreeNode

- **Approach decisions:** I kept the patches as minimal as possible since this is my first contribution to the project. The type mismatch fix is a one-line change, and the animation skip threads a single boolean parameter through the existing function chain rather than introducing any new patterns or abstractions. I also decided to commit the two fixes separately since they address different aspects of the issue and are easier to review independently.

---

## Pull Request

**PR Link:** [phpMyAdmin PR-20330](https://github.com/phpmyadmin/phpmyadmin/pull/20330)

**PR Description:** The PR explains the two fixes: the sessionStorage type mismatch that prevented tree state restoration, and the `skipTreeAnimation` parameter that prevents the tree from replaying its opening animation during restoration.

**Status:** Merged

---

## Learnings & Reflections

### Technical Skills Gained

- Learned how phpMyAdmin's server-rendered architecture works, and how the navigation tree state is persisted in sessionStorage across full page reloads.
- Got hands-on experience with JavaScript's strict equality pitfalls when dealing with the Web Storage API, which only stores strings.
- Got experience navigating a large codebase.
- Got experience using Docker to install older versions of a project (for debugging purposes).

### Challenges Overcome

- Initially didn't realize that TypeScript changes need to be compiled with `yarn build` before they take effect. Spent time wondering why my changes weren't working before figuring this out.
- The initial fix (adding `skipTreeAnimation`) didn't work on its own because the real root cause was the type mismatch preventing the restoration branch from ever running. The debug logging helped me discover that the code was hitting the wrong branch entirely.
- Navigating a large codebase for the first time was challenging. The issue referenced specific commits (36a77f7, 38b796f) which helped me locate the relevant code, but understanding how the navigation module, event-loader, CommonParams, and PHP's Header class all fit together took some tracing.

### What I'd Do Differently Next Time

- Add debug logging earlier in the process instead of assuming the fix would work from reading the code alone.
- Check for build/compile steps before spending time debugging why changes don't seem to take effect.

---

## Resources Used

- [MDN: Storage.setItem()](https://developer.mozilla.org/en-US/docs/Web/API/Storage/setItem) — Confirmed that sessionStorage coerces all values to strings.
- [GitHub Issue #18974](https://github.com/phpmyadmin/phpmyadmin/issues/18974) — The original bug report.
- [GitHub PR #18675](https://github.com/phpmyadmin/phpmyadmin/pull/18675) — Referenced by maintainers in the issue thread as potentially related.
- Commits 36a77f7 and 38b796f — Referenced by maintainers; these switched navigation links to full page reloads, which exposed the dormant type mismatch bug.
