# LRU Project Manager
*Engine rules for project position management*

## LRU Rules
1. New projects always start at position #1
2. Loading a project moves it to position #1
3. Maximum 10 active projects per type
4. Position #11 auto-archives
5. Saving a project does NOT change its position
6. Archived projects can be reloaded anytime

## Command Reference
| Command | Action |
|---------|--------|
| `new coding project [name]` | Create new project at #1 |
| `load project [name]` | Load and move to #1 |
| `save project` | Save current project progress |
| `list projects` | Show all active & archived |
| `archive project [name]` | Manually archive |

## Active Project Tracking
- Check `current-session.md` for currently active project
- Check `project-list.md` for all project positions
- Project files stored in `coding-projects/active/`
