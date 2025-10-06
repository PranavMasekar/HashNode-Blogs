---
title: "Flutter Blog Writing Style Guide (Pranav Masekar)"
slug: flutter-blog-style-guide
tags: flutter, writing, content, guidelines
---

### Purpose

This guide documents the consistent style, structure, and tone used across the Flutter blogs in this repository. Follow it to write new posts that feel native to this collection.

### Audience and Tone

- Friendly, encouraging, and action-oriented; avoid academic formality.
- Assume an intermediate Flutter developer who appreciates code-first, hands-on guidance.
- Use inclusive language: “we”, “let’s”, “you’ll”.
- Close posts with a short, upbeat sign-off like “Keep Fluttering 💙💙💙”.

### Overall Structure

1) Front matter (Hashnode-style)
   - Include: `title`, optionally `slug`, `tags`, and a cover image when publishing.

2) Intro section
   - 3–6 sentences explaining the why, context, and what the reader will build/learn.
   - Optionally list prerequisites as bullets (tools, packages, repo links).

3) Concept primer (optional but common)
   - Define key terms briefly with plain language.
   - Use short paragraphs and bolded subheads.

4) Hands-on sections (majority of the post)
   - Break into clear steps with imperative, developer-oriented headings:
     - “Let’s start with …”, “Create …”, “Implement …”, “Register …”, “Wire it into UI”.
   - Pair every step with a runnable snippet and 2–5 lines explaining what/why.
   - Keep snippets focused; prefer multiple small blocks over one giant block.

5) Recap / Wrap-up
   - Summarize what was built and any key decisions or tradeoffs.
   - Point to repo/code links if available.
   - End with a brief congratulations and signature line.

### Headings and Formatting

- Use `#` for the main title in the body only if needed; otherwise start with `###` section headers.
- Prefer short, scannable headers: “Introduction”, “Prerequisites”, “Implementation”, “Domain Layer”, “Data Layer”, “Presentation Layer”, etc.
- Use bold for emphasis and mini-subheads inline: `**What is BLoC?**`, `**GetSongsUseCase**`.
- Use ordered or unordered lists for sequences, options, and prerequisites.

### Code Snippets

- Always add language fences (```dart, ```yaml, ```bash, ```json) for syntax highlighting.
- Make snippets self-contained when possible (imports or enough context to compile/run).
- Keep lines under ~100–110 chars; split long expressions across lines for clarity.
- After a snippet, explain the key lines in bullet points or a tight paragraph. Avoid restating obvious syntax.
- Prefer multiple smaller snippets per section to illustrate progression.

### Content Patterns to Reuse

- Problem → Approach → Code → Explanation → Result.
- “What/Why” before “How”. Motivate tasks with a sentence or two.
- Show layers and flow when architectural (e.g., Clean Architecture): Presentation → Domain → Data, with arrows or bullets.
- When introducing a package, include install steps, minimal setup, and a tiny demo.

### Common Sections by Topic

- BLoC series posts
  - Intro → What is BLoC? → Minimal events/states → Bloc implementation → UI integration → Listener/Builder usage → Snackbars/Navigation demo.

- Clean Architecture posts
  - Intro → Layers overview (diagram/emoji optional) → Domain (Entities, UseCases) → Data (Models, DataSources, Repo Impl) → Presentation (Bloc + DI) → Wiring with `get_it` → Minimal UI.

- Package deep-dives (e.g., GetIt, Hive, Riverpod, Sentry)
  - Why this package → Install (pubspec) → Minimal setup → Core concepts/APIs → Small working example → Notes/caveats.

- Testing posts
  - Scope (unit/widget/integration) → Dev dependencies → Test structure (`group`, `test`, `expect`) → Mocking approach (mocktail/mockito/fakes) → 1–2 realistic examples.

### Voice and Phrasing

- Prefer direct verbs: “Create”, “Add”, “Register”, “Emit”, “Return”.
- Use short sentences and paragraphs; avoid long academic prose.
- Sprinkle small, relevant emojis for warmth (sparingly): 🧑‍💻 for hands-on, 🎉 for completion.

### Visuals and Links

- Embed diagrams or screenshots with a one-line caption when they clarify flow or results.
- Use inline links for docs, repos, and prior posts. Example: `[GetIt](https://pub.dev/packages/get_it)`.

### Examples (Templates)

#### Intro + Prerequisites

```markdown
### Introduction

State management is central to responsive Flutter apps. In this post, we’ll build a minimal BLoC, wire it to the UI, and emit states for a simple auth flow.

### Prerequisites

- flutter_bloc: ^8.x
- equatable: ^2.x
- Basic Flutter project ready
```

#### Step Section with Code

```markdown
### Create events and states

```dart
part of 'auth_bloc.dart';

abstract class AuthEvent extends Equatable {
  const AuthEvent();
  @override
  List<Object?> get props => [];
}

class LoginEvent extends AuthEvent {
  const LoginEvent({required this.email, required this.password});
  final String email;
  final String password;
}
```

Explanation:
- Base event uses Equatable to simplify equality.
- `LoginEvent` carries the credentials that the bloc will handle.
```

#### Wrap-up

```markdown
### Wrap-up

We created events, states, and a minimal BLoC, then connected it to the UI with `BlocProvider` and `BlocListener`. You can extend this foundation to real auth and navigation flows.

**Keep Fluttering 💙💙💙**
```

### Style Checklist (Before Publishing)

- Clear intro tells readers why and what.
- Prerequisites and package versions listed.
- Steps are ordered, each with runnable code and brief explanation.
- Consistent headings, bold callouts, and language fences.
- Final recap + friendly sign-off.


