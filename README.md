# 🌸 TaskFlow

**A simple and cozy productivity app to organize your tasks, track your progress, and stay focused.**

TaskFlow is a task management application built with Vue 3 and TypeScript. It provides a clean workspace where users can manage daily tasks, monitor their productivity, and use a customizable focus timer.

---

## ✨ Features

### 📋 Task Management
- Create new tasks.
- Update task titles.
- Change task status (To Do, In Progress, Completed).
- Delete tasks.
- Search tasks by title.
- Automatically save tasks using Local Storage.

### 📊 Overview Dashboard
- View the total number of tasks.
- Track tasks by status.
- Monitor overall completion progress.
- Get a quick overview of your productivity.

### ⏱️ Focus Mode
- Dedicated focus workspace.
- Customizable focus and break durations.
- Default timer: 30 minutes of focus and 10 minutes of break.
- Start, pause, and reset controls.
- Visual circular progress indicator.
- Automatic switch between focus and break sessions.

### 📈 Focus Statistics
- Track completed focus sessions.
- View daily focus time.
- Explore weekly activity.
- Keep a history of completed sessions.
- Automatically save focus history using Local Storage.

---

## 🛠️ Technologies

- **Vue 3** – Frontend framework.
- **TypeScript** – Type safety.
- **Vite** – Development server and build tool.
- **HTML5 & CSS3** – Structure and styling.
- **Local Storage** – Persistent browser storage.

---

## 🚀 Getting Started

### Prerequisites

Make sure you have installed:

- [Node.js](https://nodejs.org/)
- npm (included with Node.js)
- Git

### Installation

1. Clone the repository:

```bash
git clone https://github.com/laromia/todo.git
```

2. Navigate to the project directory:

```bash
cd todo
```

3. Install dependencies:

```bash
npm install
```

4. Start the development server:

```bash
npm run dev
```

5. Open the local URL displayed in your terminal (usually `http://localhost:5173`).

---

## 📁 Project Structure

```text
src/
├── components/
│   ├── AppHeader.vue
│   ├── AppFooter.vue
│   ├── AppSidebar.vue
│   ├── Overview.vue
│   ├── TaskForm.vue
│   ├── TaskList.vue
│   ├── TaskItem.vue
│   ├── FocusTimer.vue
│   └── FocusPage.vue
│
├── styles/
│   ├── variables.css
│   ├── global.css
│   ├── components.css
│   ├── sidebar.css
│   ├── overview.css
│   ├── tasks.css
│   └── focus.css
│
├── types/
│   └── task.ts
│
├── App.vue
└── main.ts
```

---

## 💾 Data Storage

TaskFlow uses the browser's Local Storage to save tasks and focus sessions.

Your data remains available when you refresh the page or reopen the application in the same browser.

**Note:** Data is stored locally in the browser. There is currently no backend or cloud synchronization.

---

## 🎨 Design

TaskFlow features a soft lavender and pink color palette, rounded cards, and a minimal interface designed to provide a comfortable productivity experience.

---

*Built with Vue 3, TypeScript, and a little bit of 🌸.*
