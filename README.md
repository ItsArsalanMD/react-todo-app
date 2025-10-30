# Lab Guide: React Todo List Application

## Objectives
By the end of this lab, you will:
- Build a fully functional Todo List application using React
- Understand React hooks (useState, useEffect)
- Master component composition and prop passing
- Implement CRUD operations (Create, Read, Update, Delete)
- Handle form inputs and events
- Use localStorage for data persistence

## Prerequisites
- Basic JavaScript knowledge (ES6+ features)
- Node.js installed on your system
- Basic understanding of HTML/CSS
- Text editor or IDE (VS Code recommended)

## Time Required
Approximately 2-3 hours

---

## Part 1: Project Setup (15 minutes)

### Step 1.1: Create New React Project
```bash
# Create project using Vite (faster than Create React App)
npm create vite@latest todo-app -- --template react
cd todo-app

# Install dependencies
npm install

# Start development server
npm run dev
```

### Step 1.2: Clean Up Default Files
- Delete `src/App.css` content (keep file empty for now)
- Replace `src/App.jsx` content with:

```jsx
import './App.css'

function App() {
  return (
    <div className="App">
      <h1>My Todo App</h1>
    </div>
  )
}

export default App
```

### Step 1.3: Test Setup
- Open browser to `http://localhost:5173`
- Verify "My Todo App" appears on screen

---

## Part 2: Create Basic Components (30 minutes)

### Step 2.1: Create Components Folder Structure
```
src/
├── components/
│   ├── TodoInput.jsx
│   ├── TodoList.jsx
│   ├── TodoItem.jsx
│   └── Header.jsx
├── App.jsx
├── App.css
└── main.jsx
```

### Step 2.2: Build Header Component
Create `src/components/Header.jsx`:

```jsx
const Header = () => {
  return (
    <header className="header">
      <h1>📝 My Todo List</h1>
      <p>Stay organized and productive!</p>
    </header>
  )
}

export default Header
```

### Step 2.3: Build TodoInput Component
Create `src/components/TodoInput.jsx`:

```jsx
import { useState } from 'react'

const TodoInput = ({ onAddTodo }) => {
  const [input, setInput] = useState('')

  const handleSubmit = (e) => {
    e.preventDefault()
    if (input.trim()) {
      onAddTodo(input.trim())
      setInput('')
    }
  }

  return (
    <form onSubmit={handleSubmit} className="todo-input">
      <input
        type="text"
        value={input}
        onChange={(e) => setInput(e.target.value)}
        placeholder="Add a new todo..."
        className="input-field"
      />
      <button type="submit" className="add-btn">
        Add Todo
      </button>
    </form>
  )
}

export default TodoInput
```

### Step 2.4: Build TodoItem Component
Create `src/components/TodoItem.jsx`:

```jsx
import { useState } from 'react'

const TodoItem = ({ todo, onToggle, onDelete, onUpdate }) => {
  const [isEditing, setIsEditing] = useState(false)
  const [editText, setEditText] = useState(todo.text)

  const handleUpdate = () => {
    if (editText.trim()) {
      onUpdate(todo.id, editText.trim())
    }
    setIsEditing(false)
  }

  const handleCancel = () => {
    setEditText(todo.text)
    setIsEditing(false)
  }

  return (
    <div className={`todo-item ${todo.completed ? 'completed' : ''}`}>
      <input
        type="checkbox"
        checked={todo.completed}
        onChange={() => onToggle(todo.id)}
        className="checkbox"
      />
      
      {isEditing ? (
        <div className="edit-mode">
          <input
            type="text"
            value={editText}
            onChange={(e) => setEditText(e.target.value)}
            className="edit-input"
            autoFocus
          />
          <button onClick={handleUpdate} className="save-btn">Save</button>
          <button onClick={handleCancel} className="cancel-btn">Cancel</button>
        </div>
      ) : (
        <div className="view-mode">
          <span className="todo-text">{todo.text}</span>
          <div className="actions">
            <button onClick={() => setIsEditing(true)} className="edit-btn">
              Edit
            </button>
            <button onClick={() => onDelete(todo.id)} className="delete-btn">
              Delete
            </button>
          </div>
        </div>
      )}
    </div>
  )
}

export default TodoItem
```

### Step 2.5: Build TodoList Component
Create `src/components/TodoList.jsx`:

```jsx
import TodoItem from './TodoItem'

const TodoList = ({ todos, onToggle, onDelete, onUpdate }) => {
  if (todos.length === 0) {
    return <p className="empty-state">No todos yet. Add one above!</p>
  }

  return (
    <div className="todo-list">
      {todos.map(todo => (
        <TodoItem
          key={todo.id}
          todo={todo}
          onToggle={onToggle}
          onDelete={onDelete}
          onUpdate={onUpdate}
        />
      ))}
    </div>
  )
}

export default TodoList
```

---

## Part 3: Implement State Management (45 minutes)

### Step 3.1: Update App.jsx with State Logic
Replace `src/App.jsx` content:

```jsx
import { useState, useEffect } from 'react'
import Header from './components/Header'
import TodoInput from './components/TodoInput'
import TodoList from './components/TodoList'
import './App.css'

function App() {
  const [todos, setTodos] = useState([])

  // Load todos from localStorage on component mount
  useEffect(() => {
    const savedTodos = localStorage.getItem('todos')
    if (savedTodos) {
      setTodos(JSON.parse(savedTodos))
    }
  }, [])

  // Save todos to localStorage whenever todos change
  useEffect(() => {
    localStorage.setItem('todos', JSON.stringify(todos))
  }, [todos])

  // Add new todo
  const addTodo = (text) => {
    const newTodo = {
      id: Date.now(), // Simple ID generation
      text: text,
      completed: false,
      createdAt: new Date().toISOString()
    }
    setTodos(prevTodos => [...prevTodos, newTodo])
  }

  // Toggle todo completion
  const toggleTodo = (id) => {
    setTodos(prevTodos =>
      prevTodos.map(todo =>
        todo.id === id ? { ...todo, completed: !todo.completed } : todo
      )
    )
  }

  // Delete todo
  const deleteTodo = (id) => {
    setTodos(prevTodos => prevTodos.filter(todo => todo.id !== id))
  }

  // Update todo text
  const updateTodo = (id, newText) => {
    setTodos(prevTodos =>
      prevTodos.map(todo =>
        todo.id === id ? { ...todo, text: newText } : todo
      )
    )
  }

  // Calculate stats
  const completedCount = todos.filter(todo => todo.completed).length
  const totalCount = todos.length

  return (
    <div className="App">
      <Header />
      <div className="container">
        <TodoInput onAddTodo={addTodo} />
        <div className="stats">
          {totalCount > 0 && (
            <p>{completedCount} of {totalCount} tasks completed</p>
          )}
        </div>
        <TodoList
          todos={todos}
          onToggle={toggleTodo}
          onDelete={deleteTodo}
          onUpdate={updateTodo}
        />
      </div>
    </div>
  )
}

export default App
```

---

## Part 4: Add Styling (30 minutes)

### Step 4.1: Update App.css
Replace `src/App.css` content:

```css
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  min-height: 100vh;
  color: #333;
}

.App {
  min-height: 100vh;
  padding: 20px;
}

.header {
  text-align: center;
  margin-bottom: 2rem;
  color: white;
}

.header h1 {
  font-size: 2.5rem;
  margin-bottom: 0.5rem;
}

.header p {
  font-size: 1.1rem;
  opacity: 0.9;
}

.container {
  max-width: 600px;
  margin: 0 auto;
  background: white;
  border-radius: 12px;
  padding: 2rem;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.2);
}

.todo-input {
  display: flex;
  gap: 12px;
  margin-bottom: 2rem;
}

.input-field {
  flex: 1;
  padding: 12px 16px;
  border: 2px solid #e1e5e9;
  border-radius: 8px;
  font-size: 16px;
  transition: border-color 0.2s;
}

.input-field:focus {
  outline: none;
  border-color: #667eea;
}

.add-btn {
  padding: 12px 24px;
  background: #667eea;
  color: white;
  border: none;
  border-radius: 8px;
  font-size: 16px;
  cursor: pointer;
  transition: background 0.2s;
}

.add-btn:hover {
  background: #5a6fd8;
}

.stats {
  text-align: center;
  margin-bottom: 1.5rem;
  color: #666;
  font-size: 14px;
}

.todo-list {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.todo-item {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 16px;
  border: 1px solid #e1e5e9;
  border-radius: 8px;
  transition: all 0.2s;
}

.todo-item:hover {
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

.todo-item.completed {
  opacity: 0.6;
  background: #f8f9fa;
}

.checkbox {
  width: 18px;
  height: 18px;
  cursor: pointer;
}

.view-mode {
  display: flex;
  align-items: center;
  justify-content: space-between;
  flex: 1;
}

.todo-text {
  flex: 1;
  font-size: 16px;
}

.todo-item.completed .todo-text {
  text-decoration: line-through;
}

.actions {
  display: flex;
  gap: 8px;
}

.edit-btn, .delete-btn, .save-btn, .cancel-btn {
  padding: 6px 12px;
  border-radius: 4px;
  border: none;
  cursor: pointer;
  font-size: 12px;
  transition: background 0.2s;
}

.edit-btn {
  background: #28a745;
  color: white;
}

.edit-btn:hover {
  background: #218838;
}

.delete-btn {
  background: #dc3545;
  color: white;
}

.delete-btn:hover {
  background: #c82333;
}

.edit-mode {
  display: flex;
  align-items: center;
  gap: 8px;
  flex: 1;
}

.edit-input {
  flex: 1;
  padding: 8px;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 16px;
}

.save-btn {
  background: #007bff;
  color: white;
}

.save-btn:hover {
  background: #0056b3;
}

.cancel-btn {
  background: #6c757d;
  color: white;
}

.cancel-btn:hover {
  background: #545b62;
}

.empty-state {
  text-align: center;
  color: #666;
  font-style: italic;
  padding: 2rem;
}

@media (max-width: 768px) {
  .container {
    margin: 0 10px;
    padding: 1rem;
  }
  
  .todo-input {
    flex-direction: column;
  }
  
  .actions {
    flex-direction: column;
  }
}
```

---

## Part 5: Testing and Debugging (20 minutes)

### Step 5.1: Test Basic Functionality
1. **Add todos**: Try adding several todos
2. **Toggle completion**: Check/uncheck todo items
3. **Edit todos**: Click edit, modify text, save/cancel
4. **Delete todos**: Remove individual items
5. **Persistence**: Refresh page, verify todos remain

### Step 5.2: Test Edge Cases
1. Try adding empty todos (should be prevented)
2. Add todos with only spaces (should be trimmed)
3. Edit todo to empty text (should maintain original)
4. Test with many todos (scrolling behavior)

### Step 5.3: Common Issues and Solutions

**Issue**: Todos don't persist after refresh
**Solution**: Check browser's localStorage in DevTools

**Issue**: Edit mode doesn't work properly
**Solution**: Ensure state is properly managed in TodoItem

**Issue**: Adding todos doesn't work
**Solution**: Verify onAddTodo prop is passed correctly

---

## Part 6: Enhancements (Optional - 30 minutes)

### Step 6.1: Add Filters
Add buttons to filter todos by status (All, Active, Completed):

```jsx
// Add to App.jsx
const [filter, setFilter] = useState('all')

const filteredTodos = todos.filter(todo => {
  if (filter === 'active') return !todo.completed
  if (filter === 'completed') return todo.completed
  return true
})

// Use filteredTodos instead of todos in TodoList
```

### Step 6.2: Add Clear Completed
Add button to remove all completed todos:

```jsx
const clearCompleted = () => {
  setTodos(prevTodos => prevTodos.filter(todo => !todo.completed))
}
```

### Step 6.3: Add Due Dates
Extend todo object to include due dates and add date picker.

---

## Expected Outcomes

After completing this lab, you should have:
- ✅ A fully functional Todo List application
- ✅ Understanding of React hooks (useState, useEffect)
- ✅ Knowledge of component composition and prop passing
- ✅ Experience with CRUD operations in React
- ✅ Ability to handle forms and user events
- ✅ Implementation of data persistence with localStorage

## Next Steps

1. Deploy your app to Netlify or Vercel
2. Add to your GitHub portfolio
3. Implement additional features like:
   - Drag and drop reordering
   - Categories/tags
   - Search functionality
   - Dark mode toggle
4. Consider adding React Router for multiple views
5. Explore state management with Context API or Redux

## Resources for Further Learning

- React Official Documentation: https://react.dev/
- React Hooks Guide: https://react.dev/reference/react
- localStorage MDN Documentation: https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage

---

**Congratulations!** You've built your first React application with full CRUD functionality and persistent storage.
