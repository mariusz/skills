---
name: react-code-smells-review
description: Use when reviewing React components — own code, AI-generated, or a PR diff — for maintainability anti-patterns. Checks for force-updates, direct DOM manipulation, props copied into state, uncontrolled inputs, JSX split across render helpers, prop drilling, inheritance, duplicated/oversized components, too-many-props, and low cohesion.
---

Source: [React Code Smells](https://github.com/fabiosferreira/React-Code-Smells).

Walk every changed or generated component against each row below. For each hit, quote the offending line(s), name the smell, and give the fix — don't just flag "smells present," land on a concrete diff.

## Checklist

| # | Smell | Symptom to grep for |
|---|-------|----------------------|
| 1 | Force update | `window.location.reload()`, `forceUpdate()` after a mutation |
| 2 | Direct DOM manipulation | `document.createElement`, `.appendChild`, `document.querySelector` inside a component |
| 3 | Props copied into state | `this.state = { x: props.x }` or `useState(props.x)` |
| 4 | Uncontrolled input | `<input ref={...} />` with no `value`/`onChange` |
| 5 | JSX outside render | Methods named `renderX()` called from `render()` |
| 6 | Prop drilling | A prop passed through ≥2 components that never read it |
| 7 | Inheritance over composition | `class X extends Y` where Y is another component, `super.render()` |
| 8 | Duplicated component | Two components whose JSX/logic differ only in a few values |
| 9 | Too many props | A component with 7+ individual (non-grouped) props |
| 10 | Large component | One component owns fetching, form state, list rendering, and modal logic together |
| 11 | Large file | A file exporting multiple unrelated components, 300+ lines |
| 12 | Low cohesion | A component mixing unrelated concerns (e.g. auth state + cart total) |

## 1. Force Update

**Why it's wrong:** reloading the page or calling `forceUpdate` bypasses React's data binding — state that should drive a re-render is instead thrown away and refetched from scratch, losing local state and flashing the UI.

```jsx
// Bad
onSubmitForm = async () => {
  await this.service.update({ homeDashboardId, theme, timezone, weekStart });
  window.location.reload();
};

// Good
onSubmitForm = async () => {
  const settings = await this.service.update({ homeDashboardId, theme, timezone, weekStart });
  this.setState({ settings });
};
```

## 2. Direct DOM Manipulation

**Why it's wrong:** mutating the DOM outside React's virtual DOM creates a second source of truth; the next render can silently clobber or conflict with your manual change.

```jsx
// Bad
constructor(props) {
  this.node = document.createElement('div');
  this.node.setAttribute('style', style);
  document.body.appendChild(this.node);
}

// Good
render() {
  return ReactDOM.createPortal(this.props.children, this.portalRoot);
}
```

## 3. Props in Initial State

**Why it's wrong:** state initializers run once. If the parent later passes a new prop value, the child keeps the stale copy — a bug that only shows up when the parent updates, which is easy to miss in review.

```jsx
// Bad
constructor(props) {
  this.state = { names: props.names, ages: props.ages };
}

// Good — read the prop directly, don't shadow it in state
render() {
  const { names, ages } = this.props;
  return <List names={names} ages={ages} />;
}
// If you genuinely need a derived, locally-editable copy, sync it explicitly:
// useEffect(() => setNames(props.names), [props.names])
```

## 4. Uncontrolled Components

**Why it's wrong:** reading form values via `ref` means React has no visibility into the current value, so it can't validate on keystroke, conditionally disable submit, or reset the field programmatically.

```jsx
// Bad
render() {
  return (
    <form onSubmit={this.handleSubmit}>
      <input type="text" ref={this.input} />
    </form>
  );
}

// Good
render() {
  return (
    <form onSubmit={this.handleSubmit}>
      <input type="text" value={this.state.value} onChange={this.handleChange} />
    </form>
  );
}
```

## 5. JSX Outside the Render Method

**Why it's wrong:** `renderX()` helpers are a sign the component has multiple responsibilities glued together; each one is really a separate component that can't be reused, tested, or memoized on its own.

```jsx
// Bad
renderComment() { return <div>...</div>; }
renderImage() { return <div>{this.renderComment()}</div>; }
render() { return <div>{this.renderImage()}</div>; }

// Good — extract each renderX into its own component
function Comment() { return <div>...</div>; }
function Image() { return <div><Comment /></div>; }
function Post() { return <div><Image /></div>; }
```

## 6. Prop Drilling

**Why it's wrong:** props threaded through components that never use them couple every intermediate component to the shape of data it doesn't care about — renaming a field forces edits at every hop.

```jsx
// Bad
<Gallery user={user} avatar={avatar}>
  <Image user={user} avatar={avatar}>
    <Comment user={user} avatar={avatar}>
      <Author user={user} avatar={avatar} />
    </Comment>
  </Image>
</Gallery>

// Good — pass the consumer down as a child, or use context
<Gallery>
  <Image>
    <Comment author={<Author user={user} avatar={avatar} />} />
  </Image>
</Gallery>
```

## 7. Inheritance Instead of Composition

**Why it's wrong:** extending one component from another ties them at the class level — a change to the parent's `render`/lifecycle can silently break every subclass, and the relationship is far less obvious than an explicit prop or children.

```jsx
// Bad
class Developer extends Employee {
  render() {
    return (
      <div>
        {super.render()}
        Level: {this.props.level}
      </div>
    );
  }
}

// Good
function Developer({ level, ...employeeProps }) {
  return (
    <div>
      <Employee {...employeeProps} />
      Level: {level}
    </div>
  );
}
```

## 8. Duplicated Component

**Why it's wrong:** two components that are 90% the same JSX mean every future fix or style change has to be applied twice — and reviewers/AI agents will regularly only catch one copy.

```jsx
// Bad
function PrimaryButton({ label, onClick }) {
  return <button className="btn btn-primary" onClick={onClick}>{label}</button>;
}
function DangerButton({ label, onClick }) {
  return <button className="btn btn-danger" onClick={onClick}>{label}</button>;
}

// Good
function Button({ label, onClick, variant = 'primary' }) {
  return <button className={`btn btn-${variant}`} onClick={onClick}>{label}</button>;
}
```

## 9. Too Many Props

**Why it's wrong:** a long flat prop list is hard to call correctly, hard to read at the call site, and usually signals the props are really 2-3 logical groups that belong together.

```jsx
// Bad
<UserCard name={n} age={a} email={e} phone={p} street={s} city={c} state={st} zip={z} />

// Good
<UserCard user={{ name, age, email, phone }} address={{ street, city, state, zip }} />
```

## 10. Large Component

**Why it's wrong:** a component that fetches data, owns form state, renders a list, and drives a modal is impossible to unit test in isolation and forces every contributor to load the whole thing into their head to change one part.

**Fix:** split by responsibility — one component/hook for data fetching (`useUsers()`), one for the list, one for the modal — and compose them in the parent.

## 11. Large Files

**Why it's wrong:** multiple components crammed into one 300+ line file make it hard to find a given component, and unrelated changes end up in the same diff/blame history.

**Fix:** one component (plus its tightly-coupled sub-parts) per file; re-export from an `index.ts` if you want a clean import path.

## 12. Low Cohesion

**Why it's wrong:** a component that mixes unrelated concerns — say, auth session state and cart totals — violates single responsibility: a change to one concern risks breaking the other, and the component can't be reused for just one of them.

**Fix:** split into one component per concern (`AuthStatus`, `CartSummary`) and compose them where both are needed.
