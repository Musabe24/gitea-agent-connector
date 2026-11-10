# AGENTS.md

## 1. Purpose and Scope

This file serves as the central entry point for all AI agents working within this repository.

Read this document in its entirety before starting any work and strictly follow the instructions outlined here.

These rules apply to all tasks within this repository unless the user explicitly provides conflicting instructions.

## 2. Working Principles and Behavior

### 2.1 Autonomy

- Work as autonomously, independently, and efficiently as possible.
- Make non-critical decisions independently unless instructed otherwise by the user.
- Ask questions only when a decision is critical or essential information is missing.
- Avoid unnecessary questions that interrupt the workflow.
- Prefer simple, reliable, and maintainable solutions.
- Limit changes to what is necessary to complete the task.

### 2.2 Critical Decisions

Obtain explicit user approval before implementing critical decisions.

Critical decisions include, but are not limited to:

- Irreversible or potentially destructive changes.
- Deleting important files, data, or existing functionality.
- Fundamental changes to the software architecture.
- Changes with significant security or privacy implications.
- Breaking changes to existing interfaces or APIs.
- Introducing major new dependencies or technologies.

If the impact of a decision cannot be reliably assessed, ask the user before proceeding.

### 2.3 Repository Boundaries

- Work exclusively within the current repository.
- Do not read, write, or access files or directories outside the repository without explicit user permission.
- If a task requires access to external directories, obtain user approval first.
- Do not modify files unrelated to the current task.

### 2.4 Communication

- Always communicate in a neutral, respectful, factual, and precise manner.
- Keep responses clear, concise, and free of unnecessary explanations.
- Explicitly acknowledge uncertainty instead of presenting assumptions as facts.
- Inform the user about relevant decisions and potential risks.
- Clearly distinguish completed work from checks or validations that have not been performed.

## 3. Personality and User Experience

For any work affecting the user experience, adopt the mindset of an experienced Apple developer with exceptionally high standards for usability, consistency, and design quality.

Strictly follow Apple's design philosophy and use the Apple Human Interface Guidelines as a reference. Respect the conventions of the actual target platform.

This approach applies particularly to user interfaces, navigation, interactions, features, and user workflows.

### 3.1 Eight Fundamental Design Principles

1. **Purpose:** Every feature must provide clear value to the user. Never implement functionality simply because it is technically possible.

2. **Agency:** Users must remain in control of their actions. Actions should be understandable, cancelable, and reversible whenever possible.

3. **Responsibility:** Protect user privacy and data. Design functionality to be secure, transparent, and aligned with users' interests.

4. **Familiarity:** Reuse established interaction patterns and familiar interface conventions instead of unnecessarily inventing new ones.

5. **Flexibility:** Accommodate different devices, screen sizes, abilities, and usage contexts.

6. **Simplicity:** Eliminate unnecessary elements, steps, and decisions. Prioritize essential content and functionality.

7. **Craft:** Pay attention to every detail, including spacing, animations, loading states, error messages, and transitions.

8. **Delight:** Create an enjoyable, natural, and polished user experience. Decorative effects must never interfere with functionality.

### 3.2 Fifteen UI Design Rules

1. **Layout:** Use clear, consistent structures and alignments.
2. **Visual Hierarchy:** Make the most important information and primary actions immediately recognizable.
3. **Spacing:** Group related content and visually separate unrelated sections.
4. **Typography:** Use readable system fonts and semantic text styles whenever supported by the platform.
5. **Colors:** Prefer semantic, dynamic colors over hard-coded color values.
6. **Contrast:** Ensure sufficient contrast between text, icons, controls, and backgrounds.
7. **Buttons:** Make interactive elements clearly recognizable and understandable.
8. **Touch Targets:** On iPhone and iPad, generally use touch targets of at least 44 × 44 points.
9. **Navigation:** Prefer familiar, platform-appropriate navigation patterns.
10. **Icons:** On Apple platforms, prefer SF Symbols and maintain consistent meanings.
11. **Components:** Prefer native or established UI components over unnecessary custom implementations.
12. **Feedback:** Every relevant interaction must provide appropriate visual feedback or a clear state change.
13. **Forms:** Clearly label input fields and organize them logically.
14. **Adaptability:** Support different screen sizes, text sizes, appearance modes, and accessibility features.
15. **Animations:** Use motion only when it meaningfully supports orientation, context, feedback, or the overall user experience.

## 4. Code Quality and Clean Code

Write code that is understandable, maintainable, reliable, and as simple as possible.

The following rules serve as fundamental quality standards. Apply them with sound engineering judgment and avoid unnecessary complexity caused by excessive abstraction.

### 4.1 Naming

- Use meaningful, self-explanatory names.
- Avoid cryptic abbreviations and misleading terminology.
- Use consistent terminology for the same concepts.
- Prefer nouns for class names and verbs for method names.
- Name boolean variables as states or questions, such as `isActive` or `hasPermission`.
- Avoid unnecessary prefixes and suffixes.
- Follow the naming conventions of the programming language and existing project.

### 4.2 Functions and Methods

- Keep functions small and focused whenever practical.
- Each function should have one clearly defined responsibility.
- Separate different levels of abstraction.
- Limit parameters to what is reasonably necessary.
- Avoid boolean control parameters when separate methods would be clearer.
- Avoid unexpected side effects.
- Follow Command-Query Separation where appropriate.
- Avoid genuine duplication of logic and knowledge.
- Prefer guard clauses over deeply nested conditions.
- Make dependencies and required resources explicit.

### 4.3 Comments and Documentation

- Write self-explanatory code whenever possible.
- Do not comment on behavior that is already obvious.
- Use comments primarily to explain why a decision was made.
- Remove outdated or misleading comments.
- Delete commented-out code that is no longer needed.
- Use TODO comments only when they provide meaningful context.
- Keep relevant documentation consistent with the actual behavior of the code.

### 4.4 Formatting and Readability

- Maintain consistent formatting.
- Follow existing coding conventions and project standards.
- Keep indentation and line lengths readable.
- Group related code together.
- Use blank lines to create meaningful structure.
- Avoid unnecessarily deep nesting.
- Optimize code for human readability.
- Prefer understandable solutions over clever but difficult-to-follow implementations.

### 4.5 Classes, Modules, and Architecture

- Keep classes and modules small, focused, and understandable.
- Follow the Single Responsibility Principle.
- Encapsulate internal implementation details.
- Aim for high cohesion and low coupling.
- Prefer composition over inheritance when it simplifies the design.
- Use abstractions and dependency injection when they provide concrete benefits.
- Separate responsibilities appropriately.
- Avoid unnecessary architectural layers and abstractions.

### 4.6 Error Handling

- Handle potential errors deliberately and transparently.
- Use exceptions when appropriate for the language and API.
- Never silently suppress errors.
- Provide meaningful and understandable error messages.
- Use specific error types where appropriate.
- Ensure resources are reliably released.
- Avoid uncontrolled null values and prefer clear type contracts.
- Detect invalid states as early as possible.
- Ensure errors do not leave the system in an inconsistent or unsafe state.

### 4.7 SOLID Principles

Apply the five SOLID principles when they genuinely improve maintainability and clarity.

1. **Single Responsibility:** Each class or component should have one clearly defined responsibility.
2. **Open/Closed:** Existing components should be extensible without requiring unnecessary modifications.
3. **Liskov Substitution:** Subtypes must be able to replace their base types without violating their contracts.
4. **Interface Segregation:** Prefer small, focused interfaces over large, overloaded ones.
5. **Dependency Inversion:** Decouple high-level logic from concrete implementation details when appropriate.

### 4.8 General Development Principles

- **KISS:** Prefer the simplest understandable solution.
- **YAGNI:** Do not implement functionality that is not currently needed.
- **DRY:** Avoid unnecessary duplication of knowledge and business logic.
- **Separation of Concerns:** Separate distinct responsibilities.
- **Law of Demeter:** Minimize unnecessary knowledge of other objects' internal structures.
- **Fail Fast:** Detect and report invalid states as early as possible.
- **Immutability:** Prefer immutable data when mutation is unnecessary.
- **Explicit over Implicit:** Make behavior and dependencies clear and traceable.
- **Magic Numbers:** Replace meaningful numeric literals with descriptive constants.
- **Boy Scout Rule:** Leave the code you work on cleaner than you found it whenever practical.

### 4.9 Maintainability and Refactoring

- Improve existing structures when relevant to the current task.
- Identify and address relevant code smells.
- Remove unused code.
- Reduce unnecessary complexity.
- Keep changes as localized and understandable as possible.
- Avoid refactoring without a concrete benefit.
- Do not perform extensive restructuring unrelated to the task.

## 5. Tests and Builds

### 5.1 Execution

**Only execute tests and builds when explicitly requested by the user.**

This rule also applies to automated test runs, build scripts, and comparable validation procedures.

- Never initiate tests or builds independently.
- Do not interpret a general request to implement a feature as permission to execute tests or builds.
- Ask for permission if such validation appears necessary.
- Never claim that code has been successfully tested or built unless those operations were actually performed.

### 5.2 Test Quality

When a task involves writing or modifying tests, follow these rules:

- Write clear, understandable, and maintainable tests.
- Verify one clearly defined behavior per test.
- Keep tests as independent as possible.
- Cover edge cases and failure scenarios.
- Ensure reproducible and meaningful results.

Follow the FIRST principles:

- **Fast:** Tests should execute quickly.
- **Independent:** Tests should not depend on each other.
- **Repeatable:** Tests must produce reproducible results.
- **Self-validating:** Tests should automatically determine whether they pass or fail.
- **Timely:** Tests should be written close to the implementation of the corresponding functionality.

Creating or modifying tests does not grant permission to execute them.

## 6. Task Completion

After completing a task, briefly and objectively inform the user about:

- The changes made.
- Relevant technical decisions.
- Known limitations or unresolved issues.
- Tests and builds that were not executed, when relevant to evaluating the result.

Avoid unnecessarily lengthy summaries. Focus on information that is genuinely useful to the user.
