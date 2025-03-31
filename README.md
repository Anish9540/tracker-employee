Here's the complete refactored code for your React app, following industry best practices.

---

### **1. `mockData.js` (Mock Data)**
```javascript
const initialUsers = [
    {
        id: 1,
        name: "John Doe",
        email: "john@example.com",
        password: "password123",
        role: "BOTP Employee",
        department: "Customer Support",
        joinDate: "2023-01-01",
        score: 85,
        performanceMetrics: {
            callsHandled: 100,
            customerSatisfaction: 90,
            responseTime: "5 min",
            closedTickets: 50,
        }
    },
    {
        id: 2,
        name: "Jane Smith",
        email: "jane@example.com",
        password: "manager123",
        role: "Manager",
        department: "Operations",
        joinDate: "2022-05-15"
    }
];

export default initialUsers;
```

---

### **2. `AuthContext.js` (Authentication Context)**
```javascript
import React, { createContext, useContext, useState, useEffect } from "react";
import initialUsers from "../mockData/mockData";

const AuthContext = createContext(null);

export const AuthProvider = ({ children }) => {
    const [users, setUsers] = useState(() => {
        const savedUsers = localStorage.getItem('users');
        return savedUsers ? JSON.parse(savedUsers) : initialUsers;
    });

    const [currentUser, setCurrentUser] = useState(null);

    useEffect(() => {
        localStorage.setItem('users', JSON.stringify(users));
    }, [users]);

    const login = (email, password) => {
        const user = users.find((u) => u.email === email && u.password === password);
        if (user) {
            setCurrentUser({ ...user });
        }
        return user || null;
    };

    const logout = () => setCurrentUser(null);

    const updateUserProfile = (updatedProfile) => {
        const updatedUsers = users.map(user => 
            user.id === updatedProfile.id ? { ...user, ...updatedProfile } : user
        );
        setUsers(updatedUsers);
        setCurrentUser(updatedProfile);
    };

    return (
        <AuthContext.Provider value={{ currentUser, users, login, logout, updateUserProfile }}>
            {children}
        </AuthContext.Provider>
    );
};

export const useAuth = () => useContext(AuthContext);
```

---

### **3. `ProtectedRoute.js`**
```javascript
import React from "react";
import { Navigate } from "react-router-dom";
import { useAuth } from "../context/AuthContext";

const ProtectedRoute = ({ children, requireManager = false }) => {
    const { currentUser } = useAuth();

    if (!currentUser) {
        return <Navigate to="/login" replace />;
    }

    if (requireManager && currentUser.role !== "Manager") {
        return <Navigate to="/scorecard" replace />;
    }

    return children;
};

export default ProtectedRoute;
```

---

### **4. `Navbar.js`**
```javascript
import React from "react";
import { Link, useNavigate } from "react-router-dom";
import { useAuth } from "../context/AuthContext";

const Navbar = () => {
    const { currentUser, logout } = useAuth();
    const navigate = useNavigate();

    if (!currentUser) return null;

    return (
        <nav>
            <div>
                {currentUser.role === "Manager" ? 
                    <Link to="/manager">Manager Dashboard</Link> : 
                    <Link to="/scorecard">Scorecard</Link>}
                |
                <Link to="/profile">Profile</Link>
            </div>
            <div>
                <span>{currentUser.name} ({currentUser.role})</span> |
                <button onClick={() => { logout(); navigate("/login"); }}>
                    Logout
                </button>
            </div>
        </nav>
    );
};

export default Navbar;
```

---

### **5. `RootLayout.js`**
```javascript
import React from "react";
import { Outlet } from "react-router-dom";
import Navbar from "../components/Navbar";

const RootLayout = () => (
    <div>
        <Navbar />
        <div>
            <Outlet />
        </div>
    </div>
);

export default RootLayout;
```

---

### **6. `routes.js` (Application Routes)**
```javascript
import { createBrowserRouter, Navigate } from "react-router-dom";
import RootLayout from "../layouts/RootLayout";
import LoginPage from "../pages/LoginPage";
import ProfilePage from "../pages/ProfilePage";
import ScorecardPage from "../pages/ScorecardPage";
import ManagerDashboard from "../pages/ManagerDashboard";
import EmployeeDetailsPage from "../pages/EmployeeDetailsPage";
import ErrorPage from "../pages/ErrorPage";
import ProtectedRoute from "../components/ProtectedRoute";

const createAppRouter = () => createBrowserRouter([
    {
        path: "/", element: <RootLayout />, children: [
            { path: "/", element: <Navigate to="/login" replace /> },
            { path: "login", element: <LoginPage /> },
            { path: "profile", element: <ProtectedRoute><ProfilePage /></ProtectedRoute> },
            { path: "scorecard", element: <ProtectedRoute><ScorecardPage /></ProtectedRoute> },
            { path: "manager", element: <ProtectedRoute requireManager={true}><ManagerDashboard /></ProtectedRoute> },
            { path: "employee/:id", element: <ProtectedRoute requireManager={true}><EmployeeDetailsPage /></ProtectedRoute> },
            { path: "*", element: <ErrorPage /> }
        ]
    }
]);

export default createAppRouter;
```

---

### **7. `App.js`**
```javascript
import React from "react";
import { RouterProvider } from "react-router-dom";
import { AuthProvider } from "./context/AuthContext";
import createAppRouter from "./routes/routes";

export default function App() {
    const router = createAppRouter();
    return <AuthProvider><RouterProvider router={router} /></AuthProvider>;
}
```

---

### **8. `index.js`**
```javascript
import React from "react";
import ReactDOM from "react-dom/client";
import App from "./App";

const root = ReactDOM.createRoot(document.getElementById("root"));
root.render(
    <React.StrictMode>
        <App />
    </React.StrictMode>
);
```

---

Now, your app is properly structured and follows industry standards. 🚀 Let me know if you need any modifications! 🎯
