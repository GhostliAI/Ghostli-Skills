# Ghostli AI Skills

Ready-to-use skills for Ghostli AI.

Ghostli Skills are reusable instruction folders that help Ghostli work better with specific developer workflows, technologies, frameworks, tools or project types.

Instead of writing the same long prompt every time, you can load a skill and give Ghostli a focused role, context and workflow.

Skills can be used for things like:

- debugging
- code review
- refactoring
- web development
- backend development
- API debugging
- database / SQL help
- framework-specific assistance
- game server development
- DevOps basics
- project analysis
- learning and understanding existing code

---

## What are Ghostli Skills?

A Ghostli Skill is a folder containing instructions that tell Ghostli how to behave for a specific task or development workflow.

A skill can define things like:

- what type of project Ghostli is helping with
- what problems Ghostli should focus on
- how answers should be structured
- what files, logs or errors should be checked first
- what framework, library or ecosystem rules should be followed
- how much explanation should be provided
- whether Ghostli should guide the user step by step
- what common mistakes should be avoided

Skills are useful when you want Ghostli to be more focused than a general AI coding assistant.

---

## How to use skills in Ghostli

To load a skill inside the Ghostli app, use the `/skills` command followed by the path to the skill folder.

Command:

    /skills "path/to/skill-folder"

Example on Windows:

    /skills "C:/Users/YourName/Desktop/ghostli-skills/skills/your-skill"

Example on Linux/macOS:

    /skills "/home/yourname/ghostli-skills/skills/your-skill"

After loading the skill, Ghostli will use the instructions from that folder during the conversation.

---

## How to clear the active skill

If you want to stop using the currently loaded skill, use:

    /skills clear

This will clear the active skill and return Ghostli to the default assistant behavior.

---

## Basic usage example

1. Download or clone this repository.

       git clone https://github.com/GhostliAI/ghostli-skills.git

2. Open Ghostli.

3. Load a skill using the `/skills` command.

       /skills "C:/Users/YourName/Desktop/ghostli-skills/skills/your-skill"

4. Start working with Ghostli.

Example prompt:

    Review this code and point out bugs, risky logic, security issues and possible improvements.

5. When you want to stop using the skill, clear it:

       /skills clear

---

## Recommended folder structure

Each skill should be placed in its own folder.

    ghostli-skills/
      skills/
        code-reviewer/
          README.md
          skill.md

        api-debugger/
          README.md
          skill.md

        sql-query-helper/
          README.md
          skill.md

        web-debugger/
          README.md
          skill.md

The most important file inside a skill folder is usually:

    skill.md

This file should contain the main instructions for Ghostli.

---

## Example skill structure

    skills/
      example-skill/
        README.md
        skill.md

Example `skill.md`:

    # Skill: Example Developer Helper

    You are helping the user with a specific developer workflow.

    Focus on:
    - understanding the project context
    - explaining problems clearly
    - giving practical step-by-step fixes
    - avoiding unnecessary complexity
    - asking for logs, files or error messages when needed
    - warning the user about risky changes

    When providing code:
    - explain where the code should be placed
    - mention which file should be edited
    - keep the solution practical
    - avoid rewriting the whole project unless necessary
    - explain important changes briefly

---

## Available skills

This repository will contain skills for different developer workflows.

Possible categories include:

- Code review
- Debugging
- Refactoring
- Web development
- Frontend development
- Backend development
- API debugging
- SQL and databases
- JavaScript / TypeScript
- React
- Node.js
- Python
- Lua
- DevOps basics
- Game server development
- Project structure analysis
- Documentation writing

More skills will be added over time.

---

## Creating your own skill

You can create your own Ghostli skill by making a new folder inside `skills/`.

Recommended steps:

1. Create a new folder:

       skills/my-custom-skill/

2. Add a `skill.md` file.

3. Write clear instructions for Ghostli.

4. Load it in the Ghostli app:

       /skills "path/to/skills/my-custom-skill"

Good skills should be specific.

Instead of creating a vague skill like:

    coding-helper

Create something more focused, like:

    code-reviewer
    api-debugger
    sql-query-helper
    react-debugger
    lua-error-helper
    backend-architecture-reviewer

Focused skills usually produce better answers.

---

## Best practices for writing skills

A good skill should:

- describe the target workflow clearly
- explain what Ghostli should focus on
- include common problems to check
- tell Ghostli how to format answers
- include safety or quality rules if needed
- avoid vague instructions
- stay practical and easy to use

Bad instruction:

    Help with code.

Better instruction:

    Help the user review code for bugs, risky logic, unclear structure, security issues and possible improvements. Give practical suggestions and explain which files or parts of the code should be changed.

---

## Notes

Ghostli Skills are designed to make Ghostli more useful for specific workflows.

They do not replace understanding your codebase, reading documentation or testing changes locally.

Always review AI-generated code before using it in production or on a live server.

---

## Contributing

Community contributions are welcome.

You can contribute by:

- adding new skills
- improving existing skills
- fixing unclear instructions
- adding examples
- reporting issues
- suggesting new developer workflows

If you want to add a new skill, please keep it focused, practical and easy to understand.

---

## Links

Website: https://ghostliai.eu    
Discord: https://discord.gg/Qjz5Hdxram

<p align="center">
  <img src="https://i.imgur.com/12mBWbf.png" alt="Ghostli banner" width="100%">
</p>
