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
- Use inclusive language: "we", "let's", "you'll".
- Write in natural, conversational tone with casual English expressions.
- Use relatable phrases like "trust me", "obviously!", "you know the struggle!".
- Add casual expressions like "(pain!)", "(double pain!)", "pro tip:".
- Close posts with a short, upbeat sign-off like "Keep Fluttering 💙💙💙".

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

- Use `##` for section titles in the body. Keep titles short and scannable.
- Prefer short, scannable headers: “Introduction”, “Prerequisites”, “Implementation”, “Domain Layer”, “Data Layer”, “Presentation Layer”, etc.
- Use bold for emphasis and mini-subheads inline: `**What is BLoC?**`, `**GetSongsUseCase**`.
- Use ordered or unordered lists for sequences, options, and prerequisites.

### Code Snippets

- Prefer file-referenced code blocks to cite existing repo code using this exact format:

  ```startLine:endLine:filepath
  // code content here
  ```

  - Always include start line, end line, and absolute file path within the repo.
  - Do not add language tags to file-referenced blocks.
  - Keep referenced ranges tight and focused.

- When showing commands or ad-hoc examples not in the repo, use plain fenced blocks (no language tag).
- Keep lines under ~100–110 chars; split long expressions across lines for clarity.
- After a snippet, add 2–5 bullets or a short paragraph explaining key lines or decisions.
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

- Prefer direct verbs: "Create", "Add", "Register", "Emit", "Return".
- Use short sentences and paragraphs; avoid long academic prose.
- Write in natural, conversational tone with casual English patterns.
- Use phrases like "Alright, let's get this thing working!", "First things first", "Now here's the exciting part!".
- Add relatable expressions: "obviously!", "trust me", "pro tip:", "So there you have it!".
- Include casual comments in parentheses: "(pain!)", "(double pain!)", "(obviously!)".
- Sprinkle small, relevant emojis for warmth (sparingly): 🧑‍💻 for hands-on, 🎉 for completion.

### Visuals and Links

- Embed diagrams or screenshots with a one-line caption when they clarify flow or results.
- Use inline links for docs, repos, and prior posts. Example: `[GetIt](https://pub.dev/packages/get_it)`.

### Examples (Templates)

#### Intro + Prerequisites

```markdown
### Introduction

If you're a Flutter developer, you know the struggle! State management is central to responsive Flutter apps, and in this post, we'll build a minimal BLoC, wire it to the UI, and emit states for a simple auth flow.

### Prerequisites

Before we dive in, make sure you have these things ready:

- flutter_bloc: ^8.x
- equatable: ^2.x
- A basic Flutter project to test with
- Some experience with state management (if you're reading this, you probably already have this!)
```

#### Step Section with Code

```markdown
### Create events and states

Alright, let's start with the events and states. Here's how we'll structure them:

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

What's happening here:
- Base event uses Equatable to simplify equality (no more manual comparison!)
- `LoginEvent` carries the credentials that the bloc will handle
- Pretty straightforward, right?
```

#### Wrap-up

```markdown
### Wrap-up

So there you have it! We created events, states, and a minimal BLoC, then connected it to the UI with `BlocProvider` and `BlocListener`. Trust me, once you get this foundation right, you can extend it to real auth and navigation flows without breaking a sweat.

The setup is pretty straightforward, and you'll be building complex state management in no time!

**Keep Fluttering 💙💙💙**
```

### Style Checklist (Before Publishing)

- Clear intro tells readers why and what with natural, conversational tone.
- Prerequisites and package versions listed with casual expressions.
- Steps are ordered, each with runnable code and brief explanation.
- Consistent headings, bold callouts, and language fences.
- Natural language with casual English expressions throughout.
- Casual comments and relatable phrases added appropriately.
- Final recap + friendly sign-off with "Keep Fluttering 💙💙💙".


