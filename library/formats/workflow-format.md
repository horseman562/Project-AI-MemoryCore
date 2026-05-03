# 🔄 Workflow Entry Format
*Template for process flow entries*

## Required Fields
- Workflow name and description
- Trigger/start point
- Steps with actors
- End states
- Projects using this

## Template

```markdown
# [Workflow Name]
*[What this workflow accomplishes]*

## Overview
- **Trigger**: [What starts this workflow]
- **Actors**: [Who is involved]
- **End State**: [What completion looks like]
- **Duration**: [Typical time]

## Steps

### Step 1: [Step Name]
- **Action**: [What happens]
- **Input**: [What's needed]
- **Output**: [What's produced]

### Step 2: [Step Name]
- **Action**: [What happens]
- **Input**: [What's needed]
- **Output**: [What's produced]

## Decision Points

| Condition | Path |
|-----------|------|
| [If X] | [Go to Y] |

## Error Handling
- [Error scenario]: [Recovery action]

## Projects Using This
- [Project 1]

---
*Documented: [Date]*
```

---
*Format v1.0*
