# Chapter 4: Component Composition

One of React's most powerful mental models is thinking of your application as a composition of small, independent components rather than a monolithic structure. This approach, called component composition, transforms how you architect user interfaces and is fundamental to building maintainable React applications.

## The LEGO Block Analogy

Think of React components like LEGO blocks. Each block has:

- A specific purpose and shape
- Well-defined connection points (props)
- The ability to combine with other blocks
- Independence—it doesn't need to know about other blocks to function

Just as you can build complex structures by combining simple LEGO blocks, you can build sophisticated user interfaces by composing simple React components.

```jsx
// Simple "LEGO blocks"
function Button({ children, onClick }) {
  return <button onClick={onClick}>{children}</button>;
}

function Input({ value, onChange, placeholder }) {
  return <input value={value} onChange={onChange} placeholder={placeholder} />;
}

function Card({ children }) {
  return <div className="card">{children}</div>;
}

// Composed into something more complex
function LoginForm() {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  
  return (
    <Card>
      <h2>Login</h2>
      <Input 
        value={email} 
        onChange={(e) => setEmail(e.target.value)} 
        placeholder="Email" 
      />
      <Input 
        value={password} 
        onChange={(e) => setPassword(e.target.value)} 
        placeholder="Password" 
      />
      <Button onClick={handleLogin}>
        Log In
      </Button>
    </Card>
  );
}
```

## Composition vs. Inheritance

Traditional object-oriented programming often relies on inheritance to share functionality. React favors composition instead:

```jsx
// ❌ Inheritance thinking (not React's way)
class BaseButton extends Component {
  // base button logic
}

class PrimaryButton extends BaseButton {
  // extends base with primary styling
}

class DangerButton extends BaseButton {
  // extends base with danger styling
}

// ✅ Composition thinking (React's way)
function Button({ variant = 'default', children, ...props }) {
  const className = `button button--${variant}`;
  return <button className={className} {...props}>{children}</button>;
}

// Usage
<Button variant="primary">Save</Button>
<Button variant="danger">Delete</Button>
```

The composition approach is more flexible because you can easily combine different behaviors without creating complex inheritance hierarchies.

## Breaking Down Complex UIs

The key to good component composition is learning to see complex interfaces as combinations of simpler parts. Consider a typical social media post:

```jsx
// Instead of one massive component
function SocialPost({ post }) {
  return (
    <div className="post">
      <div className="post-header">
        <img src={post.author.avatar} />
        <div>
          <h3>{post.author.name}</h3>
          <span>{post.createdAt}</span>
        </div>
        <button>⋯</button>
      </div>
      <div className="post-content">
        <p>{post.content}</p>
        {post.image && <img src={post.image} />}
      </div>
      <div className="post-actions">
        <button>👍 {post.likes}</button>
        <button>💬 {post.comments.length}</button>
        <button>↗️ Share</button>
      </div>
    </div>
  );
}

// Break it into composed components
function Avatar({ src, alt }) {
  return <img className="avatar" src={src} alt={alt} />;
}

function PostHeader({ author, createdAt, onMenuClick }) {
  return (
    <div className="post-header">
      <Avatar src={author.avatar} alt={author.name} />
      <div className="post-meta">
        <h3>{author.name}</h3>
        <time>{createdAt}</time>
      </div>
      <button onClick={onMenuClick}>⋯</button>
    </div>
  );
}

function PostContent({ content, image }) {
  return (
    <div className="post-content">
      <p>{content}</p>
      {image && <img src={image} alt="" />}
    </div>
  );
}

function PostActions({ likes, commentCount, onLike, onComment, onShare }) {
  return (
    <div className="post-actions">
      <button onClick={onLike}>👍 {likes}</button>
      <button onClick={onComment}>💬 {commentCount}</button>
      <button onClick={onShare}>↗️ Share</button>
    </div>
  );
}

function SocialPost({ post }) {
  return (
    <article className="post">
      <PostHeader 
        author={post.author} 
        createdAt={post.createdAt}
        onMenuClick={() => showPostMenu(post.id)}
      />
      <PostContent content={post.content} image={post.image} />
      <PostActions 
        likes={post.likes}
        commentCount={post.comments.length}
        onLike={() => likePost(post.id)}
        onComment={() => showComments(post.id)}
        onShare={() => sharePost(post.id)}
      />
    </article>
  );
}
```

## The Power of Children

React's `children` prop enables one of the most powerful composition patterns—creating components that wrap and enhance their contents:

```jsx
function Modal({ isOpen, onClose, children }) {
  if (!isOpen) return null;
  
  return (
    <div className="modal-overlay" onClick={onClose}>
      <div className="modal-content" onClick={e => e.stopPropagation()}>
        <button className="modal-close" onClick={onClose}>×</button>
        {children}
      </div>
    </div>
  );
}

function ConfirmationDialog({ onConfirm, onCancel }) {
  return (
    <Modal isOpen={true} onClose={onCancel}>
      <h2>Are you sure?</h2>
      <p>This action cannot be undone.</p>
      <div className="dialog-actions">
        <Button variant="secondary" onClick={onCancel}>Cancel</Button>
        <Button variant="danger" onClick={onConfirm}>Delete</Button>
      </div>
    </Modal>
  );
}
```

The `Modal` component doesn't need to know what content it's displaying—it just provides the modal behavior and styling. This makes it incredibly reusable.

## Props as Component Interfaces

Think of props as the "contract" or "interface" between components. Good prop design makes components more composable:

```jsx
// ❌ Tightly coupled - hard to reuse
function UserCard({ user }) {
  return (
    <div className="card">
      <img src={user.profilePicture} />
      <h3>{user.fullName}</h3>
      <p>{user.email}</p>
    </div>
  );
}

// ✅ Loosely coupled - highly reusable
function Card({ image, title, subtitle, children }) {
  return (
    <div className="card">
      {image && <img src={image.src} alt={image.alt} />}
      {title && <h3>{title}</h3>}
      {subtitle && <p>{subtitle}</p>}
      {children}
    </div>
  );
}

// Now it can be used for users, products, articles, etc.
<Card 
  image={{ src: user.profilePicture, alt: user.fullName }}
  title={user.fullName}
  subtitle={user.email}
/>

<Card 
  image={{ src: product.thumbnail, alt: product.name }}
  title={product.name}
  subtitle={`$${product.price}`}
>
  <Button>Add to Cart</Button>
</Card>
```

## Common Composition Patterns

### 1. Container and Presentational Components

Separate components that manage state from components that display UI:

```jsx
// Container component - manages state and logic
function TodoListContainer() {
  const [todos, setTodos] = useState([]);
  const [filter, setFilter] = useState('all');
  
  const filteredTodos = todos.filter(todo => {
    if (filter === 'active') return !todo.completed;
    if (filter === 'completed') return todo.completed;
    return true;
  });
  
  return (
    <TodoList 
      todos={filteredTodos}
      onToggle={(id) => toggleTodo(id)}
      onDelete={(id) => deleteTodo(id)}
    />
  );
}

// Presentational component - just displays data
function TodoList({ todos, onToggle, onDelete }) {
  return (
    <ul>
      {todos.map(todo => (
        <TodoItem 
          key={todo.id}
          todo={todo}
          onToggle={onToggle}
          onDelete={onDelete}
        />
      ))}
    </ul>
  );
}
```

### 2. Compound Components

Components that work together to create a cohesive interface:

```jsx
function Tabs({ children, defaultTab }) {
  const [activeTab, setActiveTab] = useState(defaultTab);
  
  return (
    <div className="tabs">
      {React.Children.map(children, child => 
        React.cloneElement(child, { activeTab, setActiveTab })
      )}
    </div>
  );
}

function TabList({ children, activeTab, setActiveTab }) {
  return (
    <div className="tab-list">
      {React.Children.map(children, (child, index) =>
        React.cloneElement(child, { 
          isActive: activeTab === index,
          onClick: () => setActiveTab(index)
        })
      )}
    </div>
  );
}

function Tab({ children, isActive, onClick }) {
  return (
    <button 
      className={`tab ${isActive ? 'active' : ''}`}
      onClick={onClick}
    >
      {children}
    </button>
  );
}

function TabPanels({ children, activeTab }) {
  return React.Children.toArray(children)[activeTab];
}

// Usage
<Tabs defaultTab={0}>
  <TabList>
    <Tab>Profile</Tab>
    <Tab>Settings</Tab>
    <Tab>Notifications</Tab>
  </TabList>
  <TabPanels>
    <ProfilePanel />
    <SettingsPanel />
    <NotificationsPanel />
  </TabPanels>
</Tabs>
```

## Benefits of Component Composition

1. **Reusability**: Small, focused components can be used in many contexts
2. **Testability**: Easier to test small components in isolation
3. **Maintainability**: Changes to one component don't affect others
4. **Readability**: Complex UIs become easier to understand when broken into named parts
5. **Collaboration**: Different team members can work on different components simultaneously

## Common Pitfalls

### 1. Over-composition

Not every piece of JSX needs to be its own component:

```jsx
// ❌ Over-composed
function Username({ name }) {
  return <span className="username">{name}</span>;
}

function UserGreeting({ user }) {
  return (
    <div>
      Hello, <Username name={user.name} />!
    </div>
  );
}

// ✅ Appropriately composed
function UserGreeting({ user }) {
  return (
    <div>
      Hello, <span className="username">{user.name}</span>!
    </div>
  );
}
```

### 2. Prop Drilling

Passing props through many layers of components that don't use them:

```jsx
// ❌ Prop drilling
function App() {
  const user = { name: 'Alice', theme: 'dark' };
  return <Layout user={user} />;
}

function Layout({ user }) {
  return <Sidebar user={user} />;
}

function Sidebar({ user }) {
  return <UserProfile user={user} />;
}

// ✅ Better approaches: Context, component composition, or state management
```

## Exercises

1. Take a complex component in your codebase and practice breaking it down into smaller, composed components.

2. Create a `Card` component that can be used for different types of content (user profiles, product listings, blog posts).

3. Build a compound component pattern for a dropdown menu or accordion.

## Summary

Component composition is about thinking in terms of building blocks rather than monolithic structures. When you master this mental model, you'll find that:

- Your components become more reusable and flexible
- Your code becomes easier to test and maintain
- Complex UIs become manageable by breaking them into simple parts
- You can build rich, interactive interfaces by combining simple components

The key is to start small, think about single responsibilities, and look for opportunities to compose larger structures from smaller, well-defined pieces. This mental model, combined with the previous concepts of declarative UI, state-driven rendering, and unidirectional data flow, forms the foundation of effective React development.