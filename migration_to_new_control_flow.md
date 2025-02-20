# Migrating Angular v17+ Control Flow

## Introduction
Angular v17 introduces a new, more performant control flow syntax that replaces traditional structural directives like `*ngIf`, `*ngFor`, and `*ngSwitch`. The new syntax improves readability, maintainability, and runtime performance.

This guide covers migrating from the old control flow syntax to the new syntax with practical examples.

## New Control Flow Syntax
Angular v17 introduces a declarative control flow with new keywords:

- `@if` (replaces `*ngIf`)
- `@for` (replaces `*ngFor`)
- `@switch` / `@case` / `@default` (replaces `*ngSwitch`)

## Migration Guide

### Replacing `*ngIf` with `@if`

#### Before (Traditional `*ngIf`)
```html
<div *ngIf="isLoggedIn">Welcome, user!</div>
```

#### After (New `@if` Syntax)
```html
@if (isLoggedIn) {
  <div>Welcome, user!</div>
}
```

### Using `@if-else`

#### Before
```html
<div *ngIf="isLoggedIn; else notLoggedIn">Welcome, user!</div>
<ng-template #notLoggedIn>
  <div>Please log in.</div>
</ng-template>
```

#### After
```html
@if (isLoggedIn) {
  <div>Welcome, user!</div>
} @else {
  <div>Please log in.</div>
}
```

---

### Replacing `*ngFor` with `@for`

#### Before
```html
<ul>
  <li *ngFor="let item of items; let i = index">{{ i }} - {{ item }}</li>
</ul>
```

#### After
```html
<ul>
  @for (item of items; track item) {
    <li>{{ item }}</li>
  }
</ul>
```

**Key Differences:**
- `track item` is used for tracking changes efficiently.
- No need for `let i = index`, but you can still use `; let i = $index` if needed.

---

### Replacing `*ngSwitch` with `@switch`

#### Before
```html
<div [ngSwitch]="status">
  <p *ngSwitchCase="'success'">Success!</p>
  <p *ngSwitchCase="'error'">Error occurred.</p>
  <p *ngSwitchDefault>Unknown status.</p>
</div>
```

#### After
```html
@switch (status) {
  @case ('success') {
    <p>Success!</p>
  }
  @case ('error') {
    <p>Error occurred.</p>
  }
  @default {
    <p>Unknown status.</p>
  }
}
```

---

## Automating Migration with VS Code Tasks

To automate the migration process for a single file in VS Code, you can set up a custom task. Add the following task configuration in `.vscode/tasks.json`:

```json
{
  "version": "2.0.0",
  "tasks": [
    {
      "type": "shell",
      "label": "Angular: Migrate html to new control flow",
      "command": "npx ng g @angular/core:control-flow --path \"${relativeFileDirname}\" --format --interactive"
    }
  ]
}
```

### Running the Task
1. Open the template file (HTML) that you want to migrate to the new control flow; it must be open to apply the task migration.
2. Open the command palette in VS Code (`Ctrl + Shift + P` on Windows/Linux, `Cmd + Shift + P` on macOS).
3. Search for `Tasks: Run Task` and select it.
4. Choose **Angular: Migrate html to new control flow** from the list.
5. Choose Continue without scanning the task output from the list.   
6. The migration will run automatically on the selected file's directory.

---

## Benefits of the New Control Flow
1. **Improved Readability**: More declarative and structured.
2. **Performance Boost**: The new syntax compiles to more efficient runtime instructions.
3. **Eliminates Extra `<ng-template>` Elements**: Simplifies markup.

## Conclusion
Migrating to Angular v17+ control flow syntax is straightforward and enhances code maintainability. The new syntax is optional but recommended for better performance and clarity. Start transitioning gradually in your codebase to leverage the improvements!

