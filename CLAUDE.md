# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a collection of Claude Code skills. Skills extend Claude's capabilities with specialized knowledge, workflows, and tool integrations. Each skill is a self-contained directory with a SKILL.md file and optional reference documentation.

**Current skills:**
- `blender-3d-printing/` - Comprehensive 3D printing workflow using Blender via MCP

## Repository Structure

```
claude-skills/
├── SKILL.md                           # Main skill definition
├── references/                        # Supporting documentation
│   ├── mcp-tools.md                  # Complete MCP tool reference
│   ├── ai-generation.md              # AI generation workflows (Hyper3D, Hunyuan3D)
│   ├── online-sources.md             # 3D print file repositories & search strategies
│   └── troubleshooting.md            # Common issues and solutions
```

## Git Workflow

This repository is tracked with git and hosted on GitHub.

**Remote:** `https://github.com/roland-hamann/claude-skills.git`

**Common commands:**
- View status: `git status`
- View commit history: `git log --oneline`
- Create commit: See git commit guidelines below
- Push changes: `git push origin master`

## Working with Skills

### Adding a New Skill

1. Create a new directory with a descriptive name (use kebab-case)
2. Create `SKILL.md` with frontmatter and skill content
3. Add supporting documentation in `references/` subdirectory if needed
4. Commit the new skill with a descriptive message

**SKILL.md frontmatter format:**
```yaml
---
name: skill-name
description: "Brief description of what the skill does and when to use it"
---
```

### Modifying an Existing Skill

1. Navigate to the skill directory
2. Edit `SKILL.md` or reference files as needed
3. Test changes if applicable
4. Commit with a clear description of what changed and why

### Skill Documentation Best Practices

Based on the existing `blender-3d-printing` skill structure:

**Progressive disclosure:**
- Start with high-level overview and key capabilities
- Provide decision trees to guide workflow selection
- Include quick reference sections for common operations
- Link to detailed reference docs for complex topics

**Reference organization:**
- `mcp-tools.md` - Complete tool reference with parameters
- `ai-generation.md` - Specialized workflow guides
- `online-sources.md` - External resource directories
- `troubleshooting.md` - Problem-solution pairs

**Code examples:**
- Include complete, runnable code snippets
- Show critical checks before operations
- Demonstrate step-by-step execution patterns
- Include validation steps after operations

## Skill Architecture Patterns

### The Blender 3D Printing Skill Pattern

The `blender-3d-printing` skill demonstrates several architectural patterns:

**1. Workflow Decision Trees**
- Search existing resources first (online repositories)
- Fall back to generation/creation if needed
- Guide users through the most efficient path

**2. Progressive Disclosure**
- Quick overview → Decision tree → Detailed workflows → Complete reference
- Main SKILL.md focuses on workflows and common tasks
- Reference docs contain exhaustive technical details

**3. Integration Layers**
- MCP tool abstraction (mcp__blender__* functions)
- Python code execution for Blender operations
- External service integration (Hyper3D, Hunyuan3D, Polyhaven, Sketchfab)

**4. Mandatory Validation Steps**
- Connection checks before operations
- 3D Print Toolbox validation before export
- Step-by-step execution with visual feedback
- Error detection and correction workflows

**5. Context-Aware Configuration**
- User-specific default profiles (Blender units, addons)
- Environment detection (API key types, modes)
- Scene setup verification at project start

## Commit Message Guidelines

Based on recent commit history:

**Format:** `[Action] [Brief description]`

**Examples:**
- `Add critical step: remove internal faces for slicer compatibility`
- `Add MakerWorld to online 3D print file sources`
- `Refactor blender-3d-printing skill with progressive disclosure`
- `Add blender-3d-printing skill`

**Good practices:**
- Start with action verb (Add, Update, Fix, Refactor, Remove)
- Be specific about what changed
- Mention the impact or reason when relevant
- Keep under 72 characters when possible

## Skills Development Philosophy

When creating or modifying skills in this repository:

**Prioritize user value:**
- Always search for existing solutions before creating new ones
- Provide clear decision trees to guide workflow selection
- Include quality evaluation criteria for external resources

**Make it practical:**
- Include complete, copy-paste ready code examples
- Show validation steps and error handling
- Provide troubleshooting guides for common issues

**Organize for discovery:**
- Use progressive disclosure (overview → details)
- Link related documentation clearly
- Separate quick reference from deep dives

**Respect context:**
- Account for user-specific configurations
- Check prerequisites before operations
- Validate state at critical checkpoints

## MCP Integration Pattern

Skills that integrate with MCP servers (like `blender-3d-printing`) follow this pattern:

**1. Connection verification**
```
mcp__[service]__get_[service]_status
```
Always verify connection before operations

**2. Tool execution**
```
mcp__[service]__[action]
```
Use service-specific MCP tools for operations

**3. Fallback execution**
```
mcp__[service]__execute_[service]_code
```
Execute arbitrary code when MCP tools don't cover the use case

**4. Step-by-step validation**
- Execute small operations incrementally
- Capture visual feedback (screenshots)
- Validate results before proceeding

## Key Technical Patterns

### Incremental Execution

**Don't:**
```python
# Large monolithic script
def prepare_model():
    # 50+ lines doing everything at once
```

**Do:**
```python
# Step 1: Select and enter mode
# Step 2: Clean geometry (one operation)
# Step 3: Validate with screenshot
# Step 4: Next operation
```

### Mandatory Validation

**Critical checkpoints in blender-3d-printing:**
- MCP connection check before any operations
- Scene unit verification at project start
- 3D Print Toolbox check before export
- Visual validation after major changes

### Resource Discovery First

**Pattern:**
1. Search existing resources (repositories, libraries)
2. Evaluate quality and suitability
3. Present findings categorized (free vs. paid)
4. Only create new if no suitable existing option

This pattern saves significant time and effort.

## Testing Changes

When modifying skills:

1. Review documentation for clarity and completeness
2. Verify code examples are syntactically correct
3. Check links to reference documentation work
4. Ensure workflow decision trees are logical
5. Test any new patterns or examples if possible

For `blender-3d-printing` specifically:
- Verify MCP tool names match current API
- Check Python code works with Blender API
- Validate workflow steps are in correct order
- Ensure mandatory checks aren't skipped
