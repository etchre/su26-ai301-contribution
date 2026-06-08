# Contribution 1: 6.0: Table tree re-opens/animates everytime + forgets position when changing inner tab

**Contribution Number:** 1

**Student:** Ethan Reed

**Issue:** https://github.com/phpmyadmin/phpmyadmin/issues/18974

**Status:** Phase I Complete

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

[Notes on setting up your local development environment - challenges you faced, how you solved them]

### Steps to Reproduce

1. [Step 1]
2. [Step 2]
3. [Observed result]

### Reproduction Evidence

- **Commit showing reproduction:** [Link to commit in your fork]
- **Screenshots/logs:** [If applicable]
- **My findings:** [What you discovered during reproduction]

---

## Solution Approach

### Analysis

[Your analysis of the root cause - what's causing the issue?]

### Proposed Solution

[High-level description of your fix approach]

### Implementation Plan

Using UMPIRE framework (adapted):

**Understand:** [Restate the problem]

**Match:** [What similar patterns/solutions exist in the codebase?]

**Plan:** [Step-by-step implementation plan]
1. [Modify file X to do Y]
2. [Add function Z]
3. [Update tests]

**Implement:** [Link to your branch/commits as you work]

**Review:** [Self-review checklist - does it follow the project's contribution guidelines?]

**Evaluate:** [How will you verify it works?]

---

## Testing Strategy

### Unit Tests

- [ ] Test case 1: [Description]
- [ ] Test case 2: [Description]
- [ ] Test case 3: [Description]

### Integration Tests

- [ ] Integration scenario 1
- [ ] Integration scenario 2

### Manual Testing

[What you tested manually and results]

---

## Implementation Notes

### Week [X] Progress

[What you built this week, challenges faced, decisions made]

### Week [Y] Progress

[Continue documenting as you work]

### Code Changes

- **Files modified:** [List]
- **Key commits:** [Links to important commits]
- **Approach decisions:** [Why you chose certain approaches]

---

## Pull Request

**PR Link:** [GitHub PR URL when submitted]

**PR Description:** [Draft or final PR description - much of the content above can be adapted]

**Maintainer Feedback:**
- [Date]: [Summary of feedback received]
- [Date]: [How you addressed it]

**Status:** [Awaiting review / Iterating / Approved / Merged]

---

## Learnings & Reflections

### Technical Skills Gained

[What you learned technically]

### Challenges Overcome

[What was hard and how you solved it]

### What I'd Do Differently Next Time

[Reflection on your process]

---

## Resources Used

- [Link to helpful documentation]
- [Tutorial or Stack Overflow post that helped]
- [GitHub issues or discussions that helped]
